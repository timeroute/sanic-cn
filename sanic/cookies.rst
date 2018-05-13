Cookies
=======

Cookies 是用户浏览器内部的一些数据。Sanic 可以读取和写入存储为键值对的 cookie 。

.. 注意::

    Cookies 能够被客户端随意修改。因此你不能只按原样在 cookies 存储登录信息，因为它们可以由
    客户自由更改。确保你存在 cookies 的数据不会被客户端伪造或篡改，使用类似 `itsdangerous`_
    来加密签名数据。


读 cookies
---------------

一个用户的 cookies 可以通过 ``Request`` 对象的 ``cookies`` 字典访问。

.. code-block:: python

    from sanic.response import text

    @app.route("/cookie")
    async def test(request):
        test_cookie = request.cookies.get('test')
        return text("Test cookie set to: {}".format(test_cookie))

写 cookies
---------------

当返回一个响应，cookies 可以在 ``Response`` 对象上设置。

.. code-block:: python

    from sanic.response import text

    @app.route("/cookie")
    async def test(request):
        response = text("There's a cookie up in this response")
        response.cookies['test'] = 'It worked!'
        response.cookies['test']['domain'] = '.gotta-go-fast.com'
        response.cookies['test']['httponly'] = True
        return response

删除 cookies
----------------

Cookies 可以语义地或明确地删除。

.. code-block:: python

    from sanic.response import text

    @app.route("/cookie")
    async def test(request):
        response = text("Time to eat some cookies muahaha")

        # This cookie will be set to expire in 0 seconds
        del response.cookies['kill_me']

        # This cookie will self destruct in 5 seconds
        response.cookies['short_life'] = 'Glad to be here'
        response.cookies['short_life']['max-age'] = 5
        del response.cookies['favorite_color']

        # This cookie will remain unchanged
        response.cookies['favorite_color'] = 'blue'
        response.cookies['favorite_color'] = 'pink'
        del response.cookies['favorite_color']

        return response

响应 cookies 可被设置成字典值并且有一下有效参数:

- ``expires`` (datetime): cookies 在客户端的浏览器上的过期时间。
- ``path`` (string): cookie 使用的子 URL。默认为 /.
- ``comment`` (string): 注释 (元数据).
- ``domain`` (string): cookie 有效的指定域名。一个明确的指定域名必须总是有 . 开始。
- ``max-age`` (number): cookie 可以存活的最大秒数。
- ``secure`` (boolean): 指定 cookie 是否只能通过 HTTPS 发送。
- ``httponly`` (boolean): 指定 cookie 是否不能被 Javascript 读取。

.. _itsdangerous: https://pythonhosted.org/itsdangerous/
