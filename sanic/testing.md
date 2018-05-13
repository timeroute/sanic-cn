# 测试

Sanic 端可以使用 `test_client` 对象进行本地测试，该对象基于额外的 [aiohttp](https://aiohttp.readthedocs.io/en/stable/)
库。

`test_client` 公开 `get`, `post`, `put`, `delete`, `patch`, `head` 和 `options` 方法供你针对你的应用程序运行。一个简单的例子 (using pytest) 如下：

```python
# Import the Sanic app, usually created with Sanic(__name__)
from external_server import app

def test_index_returns_200():
    request, response = app.test_client.get('/')
    assert response.status == 200

def test_index_put_not_allowed():
    request, response = app.test_client.put('/')
    assert response.status == 405
```

在内部，每次你调用 `test_client` 方法的任一，Sanic 会在 `127.0.0.1:42101` 执行并且你的测试请求会使用 `aiohttp` 针对你的应用程序进行执行。

`test_client` 方法接收如下参数和关键字参数：

- `uri` *(默认 `'/'`)* 一个要测试的 URI 字符串。
- `gather_request` *(默认 `True`)* 一个确定是否被函数返回原始请求的布尔值。如果设置为 `True`，返回值十一个 `(request, response)` 的元组，如果是 `False` 就只返回响应。
- `server_kwargs` *(默认 `{}`)* 一个在测试请求执行之前传入 `app.run` 的额外参数的字典。
- `debug` *(默认 `False`)* 一个确定是否在调试模式启动服务的布尔值。

程序进一步采用 `*request_args` 和 `**request_kwargs`，它们直接传递给 aiohttp ClientSession 请求。

例如，要提供数据给一个 GET 请求，你可以操作如下：

```python
def test_get_request_includes_data():
    params = {'key1': 'value1', 'key2': 'value2'}
    request, response = app.test_client.get('/', params=params)
    assert request.args.get('key1') == 'value1'
```

要提供数据给一个 JSON POST 请求：

```python
def test_post_json_request_includes_data():
    data = {'key1': 'value1', 'key2': 'value2'}
    request, response = app.test_client.post('/', data=json.dumps(data))
    assert request.json.get('key1') == 'value1'
```


更多关于 aiohttp 有效参数的信息可以在
[in the documentation for ClientSession](https://aiohttp.readthedocs.io/en/stable/client_reference.html#client-session) 找到。


## pytest-sanic

[pytest-sanic](https://github.com/yunstanford/pytest-sanic) 十一个 pytest 插件，它帮助你异步地测试你的代码。写法如下：

```python
async def test_sanic_db_find_by_id(app):
    """
    Let's assume that, in db we have,
        {
            "id": "123",
            "name": "Kobe Bryant",
            "team": "Lakers",
        }
    """
    doc = await app.db["players"].find_by_id("123")
    assert doc.name == "Kobe Bryant"
    assert doc.team == "Lakers"
```

[pytest-sanic](https://github.com/yunstanford/pytest-sanic) 还提供了一些有用的装置，如 loop, unused_port,
test_server, test_client。

```python
@pytest.yield_fixture
def app():
    app = Sanic("test_sanic_app")

    @app.route("/test_get", methods=['GET'])
    async def test_get(request):
        return response.json({"GET": True})

    @app.route("/test_post", methods=['POST'])
    async def test_post(request):
        return response.json({"POST": True})

    yield app


@pytest.fixture
def test_cli(loop, app, test_client):
    return loop.run_until_complete(test_client(app, protocol=WebSocketProtocol))


#########
# Tests #
#########

async def test_fixture_test_client_get(test_cli):
    """
    GET request
    """
    resp = await test_cli.get('/test_get')
    assert resp.status == 200
    resp_json = await resp.json()
    assert resp_json == {"GET": True}

async def test_fixture_test_client_post(test_cli):
    """
    POST request
    """
    resp = await test_cli.post('/test_post')
    assert resp.status == 200
    resp_json = await resp.json()
    assert resp_json == {"POST": True}
```
