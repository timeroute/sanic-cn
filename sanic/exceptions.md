# 异常

异常可以在请求处理程序中抛出并且自动地被 Sanic 处理。异常用一个消息作为第一个参数，同时也能携带一个状态码在 HTTP 响应中回传。

## 抛出一个异常

要抛出一个异常，只需从 `sanic.exceptions` 模块 `raise` 抛出相关的异常。

```python
from sanic.exceptions import ServerError

@app.route('/killme')
async def i_am_ready_to_die(request):
	raise ServerError("Something bad happened", status_code=500)
```

你也可以使用 `abort` 函数附上合适状态码：

```python
from sanic.exceptions import abort
from sanic.response import text

@app.route('/youshallnotpass')
async def no_no(request):
        abort(401)
        # this won't happen
        text("OK")
```

## 处理异常

要复写 Sanic 的默认异常的处理，就要用到 `@app.exception`
装饰器。该装饰器需要一个异常列表作为参数来处理。你可以传递 `SanicException` 以捕获所有异常！被装饰的异常处理程序必须带一个 `Request` 和 `Exception` 对象作为参数。

```python
from sanic.response import text
from sanic.exceptions import NotFound

@app.exception(NotFound)
async def ignore_404s(request, exception):
	return text("Yep, I totally found the page: {}".format(request.url))
```

## 有用的异常

几个最有用的异常如下所示：

- `NotFound`: 当对请求对应的路由没有找到时被调用。
- `ServerError`: 当服务器发生了某些错误是被调用。这通常在用户代码里引发异常时发生。

参考 `sanic.exceptions` 模块以获取抛出异常的完整的列表。
