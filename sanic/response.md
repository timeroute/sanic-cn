# 响应

使用 `sanic.response` 模块的函数创建响应。

## 富文本

```python
from sanic import response


@app.route('/text')
def handle_request(request):
    return response.text('Hello world!')
```

## HTML

```python
from sanic import response


@app.route('/html')
def handle_request(request):
    return response.html('<p>Hello world!</p>')
```

## JSON


```python
from sanic import response


@app.route('/json')
def handle_request(request):
    return response.json({'message': 'Hello world!'})
```

## 文件

```python
from sanic import response


@app.route('/file')
async def handle_request(request):
    return await response.file('/srv/www/whatever.png')
```

## 流

```python
from sanic import response

@app.route("/streaming")
async def index(request):
    async def streaming_fn(response):
        response.write('foo')
        response.write('bar')
    return response.stream(streaming_fn, content_type='text/plain')
```

## 文件流
对于大文件，会结合以上的 文件 和 流 实现。
```python
from sanic import response

@app.route('/big_file.png')
async def handle_request(request):
    return await response.file_stream('/srv/www/whatever.png')
```

## 重定向

```python
from sanic import response


@app.route('/redirect')
def handle_request(request):
    return response.redirect('/json')
```

## Raw

不对 body 编码的响应

```python
from sanic import response


@app.route('/raw')
def handle_request(request):
    return response.raw(b'raw data')
```

## 修改 头部 或 状态码

要修改头部或状态码，传递 `headers` 或 `status` 参数给这些函数：

```python
from sanic import response


@app.route('/json')
def handle_request(request):
    return response.json(
        {'message': 'Hello world!'},
        headers={'X-Served-By': 'sanic'},
        status=200
    )
```
