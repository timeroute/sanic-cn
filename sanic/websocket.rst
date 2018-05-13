WebSocket
=========

Sanic 支持 websockets, 建立一个 WebSocket:

.. code:: python

    from sanic import Sanic
    from sanic.response import json
    from sanic.websocket import WebSocketProtocol

    app = Sanic()

    @app.websocket('/feed')
    async def feed(request, ws):
        while True:
            data = 'hello!'
            print('Sending: ' + data)
            await ws.send(data)
            data = await ws.recv()
            print('Received: ' + data)

    if __name__ == "__main__":
        app.run(host="0.0.0.0", port=8000, protocol=WebSocketProtocol)


另外, ``app.add_websocket_route`` 方法能被用来替换装饰器:

.. code:: python

    async def feed(request, ws):
        pass

    app.add_websocket_route(feed, '/feed')


一个 Websocket 路由的处理程序传递请求作为第一个参数，并且一个 Websocket 协议对象作为第二个
参数。该协议对象有 ``send`` 和 ``recv`` 方法来分别发送和接收数据。


你可以通过 ``app.config`` 建立你自己的 WebSocket 配置, 就像

.. code:: python
    app.config.WEBSOCKET_MAX_SIZE = 2 ** 20
    app.config.WEBSOCKET_MAX_QUEUE = 32
    app.config.WEBSOCKET_READ_LIMIT = 2 ** 16
    app.config.WEBSOCKET_WRITE_LIMIT = 2 ** 16

想要了解更多请去 ``Configuration`` 章节。
