# 蓝图

蓝图 (Blueprints) 是用在一个应用程序里可用作子路由的对象。作为添加路由到应用程序实例的替代者，蓝图为添加路由定义了相似的方法，即用一种灵活而且可插拔的方法注册到应用程序实例。

蓝图对于大型应用特别有用，当你的应用逻辑能被分解成若干个组或者责任区域。

## 我的第一个蓝图

以下展示了一个非常简单的蓝图，即在你的应用的根 url `/` 注册了一个处理程序。

假设你保存了这个一会儿能被 import 到你的主应用的文件 `my_blueprint.py`。

```python
from sanic.response import json
from sanic import Blueprint

bp = Blueprint('my_blueprint')

@bp.route('/')
async def bp_root(request):
    return json({'my': 'blueprint'})

```

## 注册蓝图

蓝图必须被应用程序注册。

```python
from sanic import Sanic
from my_blueprint import bp

app = Sanic(__name__)
app.blueprint(bp)

app.run(host='0.0.0.0', port=8000, debug=True)
```

这将会添加蓝图到应用程序并且注册任意的被蓝图定义的路由。在 `app.router` 里已经注册的路由看起来如下：

```python
[Route(handler=<function bp_root at 0x7f908382f9d8>, methods=None, pattern=re.compile('^/$'), parameters=[])]
```

## 蓝图 组 和 嵌套

蓝图也可能作为一个列表或元组的部分进行注册，其中注册员将递归地遍历蓝图的任何子序列并相应地注册它们。`Blueprint.group` 方法提供简化的程序，允许一个 '模拟' 后端目录结构模仿从前端看到的东西。考虑这个 (颇有认为的) 例子：

```
api/
├──content/
│  ├──authors.py
│  ├──static.py
│  └──__init__.py
├──info.py
└──__init__.py
app.py
```

应用的蓝图的初始化层次结构如下所示：

```python
# api/content/authors.py
from sanic import Blueprint

authors = Blueprint('content_authors', url_prefix='/authors')
```
```python
# api/content/static.py
from sanic import Blueprint

static = Blueprint('content_static', url_prefix='/static')
```
```python
# api/content/__init__.py
from sanic import Blueprint

from .static import static
from .authors import authors

content = Blueprint.group(assets, authors, url_prefix='/content')
```
```python
# api/info.py
from sanic import Blueprint

info = Blueprint('info', url_prefix='/info')
```
```python
# api/__init__.py
from sanic import Blueprint

from .content import content
from .info import info

api = Blueprint.group(content, info, url_prefix='/api')
```

在 `app.py` 中注册这些蓝图现在可以这样完成：

```python
# app.py
from sanic import Sanic

from .api import api

app = Sanic(__name__)

app.blueprint(api)
```

## 使用蓝图

蓝图具有与应用程序实例相同的功能。

### WebSocket 路由

WebSocket 处理程序能够使用 `@bp.websocket` 装饰器或者 `bp.add_websocket_route` 方法在蓝图注册。

### 中间件

使用蓝图允许你可以全局地注册中间件。

```python
@bp.middleware
async def print_on_request(request):
	print("I am a spy")

@bp.middleware('request')
async def halt_request(request):
	return text('I halted the request')

@bp.middleware('response')
async def halt_response(request, response):
	return text('I halted the response')
```

### 异常

异常可以被专门用于全局的蓝图。

```python
@bp.exception(NotFound)
def ignore_404s(request, exception):
	return text("Yep, I totally found the page: {}".format(request.url))
```

### 静态文件

静态文件可以在蓝图前缀下全局提供。

```python

# suppose bp.name == 'bp'

bp.static('/web/path', '/folder/to/serve')
# also you can pass name parameter to it for url_for
bp.static('/web/path', '/folder/to/server', name='uploads')
app.url_for('static', name='bp.uploads', filename='file.txt') == '/bp/web/path/file.txt'

```

## 启动和停止

蓝图可以在服务器启动和停止程序期间运行程序。如果在多进程模式 (多于一个 worker) 运行，他们会在 workers fork 之后被触发。

有效的事件如下:

- `before_server_start`: 在服务器开始接收连接之前执行
- `after_server_start`: 在服务器开始接收连接之后执行
- `before_server_stop`: 在服务器停止接收连接之前执行
- `after_server_stop`: 在服务器停止并且所有请求已经问你成之后执行

```python
bp = Blueprint('my_blueprint')

@bp.listener('before_server_start')
async def setup_connection(app, loop):
    global database
    database = mysql.connect(host='127.0.0.1'...)

@bp.listener('after_server_stop')
async def close_connection(app, loop):
    await database.close()
```

## 用例: API 版本

蓝图能很好的用于 API 版本，一个蓝图可以指向 `/v1/<routes>`，另一个指向 `/v2/<routes>`。

当一个蓝图初始化了，它可以带一个可选的 `url_prefix` 参数，该参数将被添加到所有在蓝图定义的路由中。这个功能可以被用来执行我们的 API 版本方案。

```python
# blueprints.py
from sanic.response import text
from sanic import Blueprint

blueprint_v1 = Blueprint('v1', url_prefix='/v1')
blueprint_v2 = Blueprint('v2', url_prefix='/v2')

@blueprint_v1.route('/')
async def api_v1_root(request):
    return text('Welcome to version 1 of our documentation')

@blueprint_v2.route('/')
async def api_v2_root(request):
    return text('Welcome to version 2 of our documentation')
```

当我们在 app 上注册了我们的蓝图，这些路由 `/v1` 和 `/v2` 现在将指向独立的蓝图，允许为每一个 API 版本创建 *子站点*

```python
# main.py
from sanic import Sanic
from blueprints import blueprint_v1, blueprint_v2

app = Sanic(__name__)
app.blueprint(blueprint_v1, url_prefix='/v1')
app.blueprint(blueprint_v2, url_prefix='/v2')

app.run(host='0.0.0.0', port=8000, debug=True)
```

## 用 `url_for` 建立 URL

如果你希望为蓝图里的路由生成 URL，记住端点命名带着 `<blueprint_name>.<handler_name>` 格式。例如：

```python
@blueprint_v1.route('/')
async def root(request):
    url = request.app.url_for('v1.post_handler', post_id=5) # --> '/v1/post/5'
    return redirect(url)


@blueprint_v1.route('/post/<post_id>')
async def post_handler(request, post_id):
    return text('Post {} in Blueprint V1'.format(post_id))
```
