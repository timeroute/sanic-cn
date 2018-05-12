# Routing

路由允许用户为不同的 URL 端指定处理函数。

一个基本的路由看起来如下，其中 `app` 是一个
`Sanic` 类的实例：

```python
from sanic.response import json

@app.route("/")
async def test(request):
    return json({ "hello": "world" })
```

当 `http://server.url/` 这个 url 被访问 (服务器的根 url), 最后的 `/` 被路由匹配到 `test` 这个返回一个 JSON 对象的处理函数。

Sanic 处理函数必须使用 `async def` 语法来定义，因为他们是异步函数。

## 请求参数

Sanic 附带一个支持请求参数的基本路由。

要指定一个请求参数，用尖括号包裹如下： `<PARAM>`。请求参数作为关键字被传递到处理函数。

```python
from sanic.response import text

@app.route('/tag/<tag>')
async def tag_handler(request, tag):
	return text('Tag - {}'.format(tag))
```

要指定一个参数的类型，需在括号里定义的参数名后面加一个 `:type` 。如果参数匹配不到这个指定的类型， Sanic 将会抛出一个 `NotFound` 异常，在 URL 上以一个 `404: Page not found` 错误作为结果。

```python
from sanic.response import text

@app.route('/number/<integer_arg:int>')
async def integer_handler(request, integer_arg):
	return text('Integer - {}'.format(integer_arg))

@app.route('/number/<number_arg:number>')
async def number_handler(request, number_arg):
	return text('Number - {}'.format(number_arg))

@app.route('/person/<name:[A-z]+>')
async def person_handler(request, name):
	return text('Person - {}'.format(name))

@app.route('/folder/<folder_id:[A-z0-9]{0,4}>')
async def folder_handler(request, folder_id):
	return text('Folder - {}'.format(folder_id))

```

## HTTP 请求方法

默认情况下，在一个 URL 上定义一个路由将只会对 GET 请求有效。然而， `@app.route` 装饰器接受一个可选的参数， `methods`，该参数允许处理函数处理 HTTP 请求方法列表中的任意一个方法。

```python
from sanic.response import text

@app.route('/post', methods=['POST'])
async def post_handler(request):
	return text('POST request - {}'.format(request.json))

@app.route('/get', methods=['GET'])
async def get_handler(request):
	return text('GET request - {}'.format(request.args))

```

还有一个可选的 `host` 参数 (可以是一个 list 或 string)。这限定了一个路由到提供的主机或主机组。如果一个路由没有 host，这会设置默认。

```python
@app.route('/get', methods=['GET'], host='example.com')
async def get_handler(request):
	return text('GET request - {}'.format(request.args))

# if the host header doesn't match example.com, this route will be used
@app.route('/get', methods=['GET'])
async def get_handler(request):
	return text('GET request in default - {}'.format(request.args))
```

还有简写方法装饰器：

```python
from sanic.response import text

@app.post('/post')
async def post_handler(request):
	return text('POST request - {}'.format(request.json))

@app.get('/get')
async def get_handler(request):
	return text('GET request - {}'.format(request.args))

```
## `add_route` 方法

正如我们看到的，路由经常由 `@app.route` 装饰器来指定。然而，这个装饰器其实只是一个 `app.add_route`
方法的包装，用法如下：

```python
from sanic.response import text

# Define the handler functions
async def handler1(request):
	return text('OK')

async def handler2(request, name):
	return text('Folder - {}'.format(name))

async def person_handler2(request, name):
	return text('Person - {}'.format(name))

# Add each handler function as a route
app.add_route(handler1, '/test')
app.add_route(handler2, '/folder/<name>')
app.add_route(person_handler2, '/person/<name:[A-z]>', methods=['GET'])
```

## URL 由 `url_for` 创建

Sanic 提供了一个 `url_for` 方法来生成一组基于这个处理方法名称的 URL。如果你想在 app 中避免硬编码 url，这是有用的。相反，你可以只参考这个处理名称。举个例子：

```python
from sanic.response import redirect

@app.route('/')
async def index(request):
    # generate a URL for the endpoint `post_handler`
    url = app.url_for('post_handler', post_id=5)
    # the URL is `/posts/5`, redirect to it
    return redirect(url)    

@app.route('/posts/<post_id>')
async def post_handler(request, post_id):
    return text('Post - {}'.format(post_id))
```

在使用 `url_for` 时需要注意的其他事项：

- 关键字参数传递给 `url_for` 不是请求参数将会被包括在 URL 的请求参数里。例如：
```python
url = app.url_for('post_handler', post_id=5, arg_one='one', arg_two='two')
# /posts/5?arg_one=one&arg_two=two
```
- 多元的参数也会被 `url_for` 传递。例如：
```python
url = app.url_for('post_handler', post_id=5, arg_one=['one', 'two'])
# /posts/5?arg_one=one&arg_one=two
```
- 还有一些特殊的参数 (`_anchor`, `_external`, `_scheme`, `_method`, `_server`) 传递到 `url_for` 会有一个指定的 url 建立 (`_method` 现在不被支持并且会被忽略)。例如：
```python
url = app.url_for('post_handler', post_id=5, arg_one='one', _anchor='anchor')
# /posts/5?arg_one=one#anchor

url = app.url_for('post_handler', post_id=5, arg_one='one', _external=True)
# //server/posts/5?arg_one=one
# _external requires passed argument _server or SERVER_NAME in app.config or url will be same as no _external

url = app.url_for('post_handler', post_id=5, arg_one='one', _scheme='http', _external=True)
# http://server/posts/5?arg_one=one
# when specifying _scheme, _external must be True

# you can pass all special arguments one time
url = app.url_for('post_handler', post_id=5, arg_one=['one', 'two'], arg_two=2, _anchor='anchor', _scheme='http', _external=True, _server='another_server:8888')
# http://another_server:8888/posts/5?arg_one=one&arg_one=two&arg_two=2#anchor
```
- 所有有效的参数必须被传递到 `url_for` 用来建立一个 URL。如果一个参数没有提供，或者没有匹配到指定的类型，会抛出一个 `URLBuildError`。

## WebSocket 路由

对于 WebSocket 协议的路由可以使用 `@app.websocket`
装饰器定义：

```python
@app.websocket('/feed')
async def feed(request, ws):
    while True:
        data = 'hello!'
        print('Sending: ' + data)
        await ws.send(data)
        data = await ws.recv()
        print('Received: ' + data)
```

另外, `app.add_websocket_route` 方法可以用来替代这个装饰器：

```python
async def feed(request, ws):
    pass

app.add_websocket_route(my_websocket_handler, '/feed')
```

一个 WebSocket 路由的处理程序作为第一个参数传递请求，同时一个 WebSocket 协议对象作为第二个参数。这个协议对象有 `send`
和 `recv` 方法分别发送和接受数据。

支持 WebSocket 需要 Aymeric Augustin 提供的 [websockets](https://github.com/aaugustin/websockets)
包。

## 关于 `strict_slashes`

你可以设计严格的 `routes` 来决定要不要斜线。它是可配置的。

```python

# provide default strict_slashes value for all routes
app = Sanic('test_route_strict_slash', strict_slashes=True)

# you can also overwrite strict_slashes value for specific route
@app.get('/get', strict_slashes=False)
def handler(request):
    return text('OK')

# It also works for blueprints
bp = Blueprint('test_bp_strict_slash', strict_slashes=True)

@bp.get('/bp/get', strict_slashes=False)
def handler(request):
    return text('OK')

app.blueprint(bp)
```

## 用户定义路由命名

你可以传递 `name` 来改变路由命名以避免使用默认命名  (`handler.__name__`)。

```python

app = Sanic('test_named_route')

@app.get('/get', name='get_handler')
def handler(request):
    return text('OK')

# then you need use `app.url_for('get_handler')`
# instead of # `app.url_for('handler')`

# It also works for blueprints
bp = Blueprint('test_named_bp')

@bp.get('/bp/get', name='get_handler')
def handler(request):
    return text('OK')

app.blueprint(bp)

# then you need use `app.url_for('test_named_bp.get_handler')`
# instead of `app.url_for('test_named_bp.handler')`

# different names can be used for same url with different methods

@app.get('/test', name='route_test')
def handler(request):
    return text('OK')

@app.post('/test', name='route_post')
def handler2(request):
    return text('OK POST')

@app.put('/test', name='route_put')
def handler3(request):
    return text('OK PUT')

# below url are the same, you can use any of them
# '/test'
app.url_for('route_test')
# app.url_for('route_post')
# app.url_for('route_put')

# for same handler name with different methods
# you need specify the name (it's url_for issue)
@app.get('/get')
def handler(request):
    return text('OK')

@app.post('/post', name='post_handler')
def handler(request):
    return text('OK')

# then
# app.url_for('handler') == '/get'
# app.url_for('post_handler') == '/post'
```

## 为静态资源建立 URL

现在你可以使用 `url_for` 给静态资源建立 url。如果是为文件直传的， `filename` 会被忽略。

```python

app = Sanic('test_static')
app.static('/static', './static')
app.static('/uploads', './uploads', name='uploads')
app.static('/the_best.png', '/home/ubuntu/test.png', name='best_png')

bp = Blueprint('bp', url_prefix='bp')
bp.static('/static', './static')
bp.static('/uploads', './uploads', name='uploads')
bp.static('/the_best.png', '/home/ubuntu/test.png', name='best_png')
app.blueprint(bp)

# then build the url
app.url_for('static', filename='file.txt') == '/static/file.txt'
app.url_for('static', name='static', filename='file.txt') == '/static/file.txt'
app.url_for('static', name='uploads', filename='file.txt') == '/uploads/file.txt'
app.url_for('static', name='best_png') == '/the_best.png'

# blueprint url building
app.url_for('static', name='bp.static', filename='file.txt') == '/bp/static/file.txt'
app.url_for('static', name='bp.uploads', filename='file.txt') == '/bp/uploads/file.txt'
app.url_for('static', name='bp.best_png') == '/bp/static/the_best.png'

```
