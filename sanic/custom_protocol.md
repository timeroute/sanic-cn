# 自定义协议

*注意：这是高级用户，大部分读者用不到这功能。*

你可以通过指定一个继承于 [asyncio.protocol](https://docs.python.org/3/library/asyncio-protocol.html#protocol-classes) 的自定义的协议来修改 Sanic 协议的动作。该协议就可以作为关键字参数 `protocol` 被传递到 `sanic.run` 方法。

自定义协议类的构造函数从 Sanic 接收一下关键字参数。

- `loop`: 一个兼容 `asyncio` 的事件循环。
- `connections`: 一个保存协议对象的 `set`。当 Sanic 接收到
  `SIGINT` 或 `SIGTERM`，会为所有保存在集合里协议执行 `protocol.close_if_idle`。
- `signal`: 一个带有 `stopped` 属性的 `sanic.server.Signal` 对象。当 Sanic 接收到 `SIGINT` 或 `SIGTERM`，`signal.stopped` 被置为 `True`。
- `request_handler`: 一个带着 `sanic.request.Request` 对象和 `response` 回调作为参数的协程。
- `error_handler`: 一个当异常抛出时回调的 `sanic.exceptions.Handler`。
- `request_timeout`: 请求超时秒数。
- `request_max_size`: 一个请求指定的最大数值，用 bytes。

## 例子

如果一个处理程序不返回 `HTTPResponse` 对象就会在默认协议引发错误。

通过复写 `write_response` 协议方法，当一个处理程序返回一个字符串它将转换成一个 `HTTPResponse` 对象。

```python
from sanic import Sanic
from sanic.server import HttpProtocol
from sanic.response import text

app = Sanic(__name__)


class CustomHttpProtocol(HttpProtocol):

    def __init__(self, *, loop, request_handler, error_handler,
                 signal, connections, request_timeout, request_max_size):
        super().__init__(
            loop=loop, request_handler=request_handler,
            error_handler=error_handler, signal=signal,
            connections=connections, request_timeout=request_timeout,
            request_max_size=request_max_size)

    def write_response(self, response):
        if isinstance(response, str):
            response = text(response)
        self.transport.write(
            response.output(self.request.version)
        )
        self.transport.close()


@app.route('/')
async def string(request):
    return 'string'


@app.route('/1')
async def response(request):
    return text('response')

app.run(host='0.0.0.0', port=8000, protocol=CustomHttpProtocol)
```
