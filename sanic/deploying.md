# 部署

部署 Sanic 由内建的 web 服务器简化。在定义一个 `sanic.Sanic` 实例后，我们可以使用如下关键字参数调用 `run` 方法：

- `host` *(默认 `"127.0.0.1"`)*: 主机服务器地址。
- `port` *(默认 `8000`)*: 主机服务器端口。
- `debug` *(默认 `False`)*: 启动调试输出 (减缓服务器速度).
- `ssl` *(默认 `None`)*: 为工作进程的 SSL 加密的 `SSLContext`。
- `sock` *(默认 `None`)*: 服务器接受连接的套接字。
- `workers` *(默认 `1`)*: 产生的工作进程的数量。
- `loop` *(默认 `None`)*: 一个兼容 `asyncio` 的事件循环。如果 none 被指定，Sanic 会建立自己的事件循环。
- `protocol` *(默认 `HttpProtocol`)*: [asyncio.protocol](https://docs.python.org/3/library/asyncio-protocol.html#protocol-classes) 的继承。

## 工作进程

默认情况下，Sanic 使用一个 CPU 核心在主程序中监听。为了加速，只需在 `run` 参数里指定 workers 工作进程的数量。

```python
app.run(host='0.0.0.0', port=1337, workers=4)
```

Sanic 会自动启动多个进程并且在它们之间路由流量。我们建议尽可能多得根据你有的 CPU 核心数量。

## 在终端运行

如果你喜欢使用命令行参数，你可以通过执行这个模块启动一个 Sanic 服务。例如，如果你在一个名叫 `server.py` 文件里初始化了一个叫做 `app` 的 Sanic 应用，你可以运行服务如下：

`python -m sanic server.app --host=0.0.0.0 --port=1337 --workers=4`

这种运行 sanic 的方式，不需要在你的 Python 文件中调用 `app.run`。如果你这样做了，确保包装了它以便它只在解释器运行时执行。

```python
if __name__ == '__main__':
    app.run(host='0.0.0.0', port=1337, workers=4)
```

## 在 Gunicorn 运行

[Gunicorn](http://gunicorn.org/) ‘Green Unicorn’ 是一个 UNIX 下的 WSGI HTTP 服务器。这是一个从 Ruby 的 Unicorn 项目移植过来的预分叉工作者模型。

为了用 Gunicorn 运行 Sanic 应用程序，你需要为 Gunicorn `worler-class` 参数使用特定的 `sanic.worker.GunicornWorker`：

```
gunicorn myapp:app --bind 0.0.0.0:1337 --worker-class sanic.worker.GunicornWorker
```

如果你的应用程序遭受内存泄漏，你可以将 Gunicorn 配置为在处理完指定数量的请求后正常重启工作程序。这是一种便利的方法来帮助限制内存泄漏的影响。

查看 [Gunicorn Docs](http://docs.gunicorn.org/en/latest/settings.html#max-requests) 获取更多信息。

## 异步支持
如果你 *需要* 分享 sanic 进程给其他应用程序，特别是 `loop`，这非常合适。但是请注意，此方法不支持使用多个进程，并且一般来说这不是最佳运行应用的方式。

这是一个不完整的例子 (请参考示例中的 `run_async.py` 以获取更实用的内容)：

```python
server = app.create_server(host="0.0.0.0", port=8000)
loop = asyncio.get_event_loop()
task = asyncio.ensure_future(server)
loop.run_forever()
```
