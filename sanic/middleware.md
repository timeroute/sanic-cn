# 中间件 和 监听器

中间件是在请求到服务器前或后执行的程序。它们能用来修改请求到用户定义的处理程序或者从用户定义的处理程序中响应。

另外，Sanic 提供的监听器允许你在应用程序生命周期的多个端运行代码

## 中间件

有两种中间件：请求和响应。两者都使用 `@app.middleware` 装饰器声明，装饰器的参数是一个表示类型的字符串：`'request'` 或 `'response'`.

* 请求中间件接收仅作为 `request` 的参数。
* 响应中间件接收同时可以是 `request` 和 `response` 的参数。

最简单的中间件根本不会修改请求和响应：

```python
@app.middleware('request')
async def print_on_request(request):
	print("I print when a request is received by the server")

@app.middleware('response')
async def print_on_response(request, response):
	print("I print when a response is returned by the server")
```

## 修改请求或响应

中间件可以修改给定的请求或响应的参数，*只要它们不返回*。下面的例子显示了这个实际的用例：

```python
app = Sanic(__name__)

@app.middleware('response')
async def custom_banner(request, response):
	response.headers["Server"] = "Fake-Server"

@app.middleware('response')
async def prevent_xss(request, response):
	response.headers["x-xss-protection"] = "1; mode=block"

app.run(host="0.0.0.0", port=8000)
```

以上代码会按顺序接收两个中间件。第一个中间件 **custom_banner** 会修改 HTTP 响应头 *Server* 为 *Fake-Server*，第二个中间件 **prevent_xss** 会添加 HTTP
头以避免跨站脚本 (XSS) 攻击。这两个程序在用户程序返回响应后调用。

## 提前响应

如果中间件返回一个 `HTTPResponse` 对象，该请求会停止处理并且响应会被返回。如果这发生在一个到达相应用户路由处理程序前的请求，该处理程序永远不会被调用。返回响应也会阻止任何进一步的中间件运行。

```python
@app.middleware('request')
async def halt_request(request):
	return text('I halted the request')

@app.middleware('response')
async def halt_response(request, response):
	return text('I halted the response')
```

## 监听器

如果你想执行 startup/teardown 代码作为你服务应用的启动或关闭，你可以使用以下监听器：

- `before_server_start`
- `after_server_start`
- `before_server_stop`
- `after_server_stop`

这些监听器
These listeners are implemented as decorators on functions which accept the app object as well as the asyncio loop.

For example:

```python
@app.listener('before_server_start')
async def setup_db(app, loop):
    app.db = await db_setup()

@app.listener('after_server_start')
async def notify_server_started(app, loop):
    print('Server successfully started!')

@app.listener('before_server_stop')
async def notify_server_stopping(app, loop):
    print('Server shutting down!')

@app.listener('after_server_stop')
async def close_db(app, loop):
    await app.db.close()
```

It's also possible to register a listener using the `register_listener` method.
This may be useful if you define your listeners in another module besides
the one you instantiate your app in.

```python
app = Sanic()

async def setup_db(app, loop):
    app.db = await db_setup()

app.register_listener(setup_db, 'before_server_start')

```

If you want to schedule a background task to run after the loop has started,
Sanic provides the `add_task` method to easily do so.

```python
async def notify_server_started_after_five_seconds():
    await asyncio.sleep(5)
    print('Server successfully started!')

app.add_task(notify_server_started_after_five_seconds())
```

Sanic will attempt to automatically inject the app, passing it as an argument to the task:

```python
async def notify_server_started_after_five_seconds(app):
    await asyncio.sleep(5)
    print(app.name)

app.add_task(notify_server_started_after_five_seconds)
```

Or you can pass the app explicitly for the same effect:

```python
async def notify_server_started_after_five_seconds(app):
    await asyncio.sleep(5)
    print(app.name)

app.add_task(notify_server_started_after_five_seconds(app))
`
