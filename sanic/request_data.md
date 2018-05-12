# 请求数据

当一端接收一个 HTTP 请求，路由函数会传递一个 `Request` 对象。

下面的变量作为 `Request` 对象的属性是可访问的：

- `json` (任意类型) - JSON body

  ```python
  from sanic.response import json

  @app.route("/json")
  def post_json(request):
      return json({ "received": True, "message": request.json })
  ```

- `args` (字典) - 请求参数的变量。 一个请求参数就是一个类似 `?key1=value1&key2=value2` URL 的一部分。如果都被解析了，那么 `args` 字典数据就是 `{'key1': ['value1'], 'key2': ['value2']}`。
  这个请求的 `query_string` 变量保存了未被解析的字符数据。

  ```python
  from sanic.response import json

  @app.route("/query_string")
  def query_string(request):
      return json({ "parsed": True, "args": request.args, "url": request.url, "query_string": request.query_string })
  ```

- `raw_args` (字典) - 很多情况你需要访问一个较少包装的字典的 url 参数。对于同样上一个 URL `?key1=value1&key2=value2`，  `raw_args` 字典会是 `{'key1': 'value1', 'key2': 'value2'}`。

- `files` (`File` 对象的字典) - 文件列表，包括 name, body, 和 type

  ```python
  from sanic.response import json

  @app.route("/files")
  def post_json(request):
      test_file = request.files.get('test')

      file_parameters = {
          'body': test_file.body,
          'name': test_file.name,
          'type': test_file.type,
      }

      return json({ "received": True, "file_names": request.files.keys(), "test_file_parameters": file_parameters })
  ```

- `form` (字典) - 提交 form 变量。

  ```python
  from sanic.response import json

  @app.route("/form")
  def post_json(request):
      return json({ "received": True, "form_data": request.form, "test": request.form.get('test') })
  ```

- `body` (bytes) - 提交原始请求体数据。这个属性允许请求的原始数据的获取而不管内容的类型。

  ```python
  from sanic.response import text

  @app.route("/users", methods=["POST",])
  def create_user(request):
      return text("You are trying to create a user with the following POST: %s" % request.body)
  ```

- `headers` (字典) - 一个例子-不严格区分的包含请求头部的字典。

- `method` (字符) - HTTP 请求方法 (ie `GET`, `POST`)。

- `ip` (字符) - 请求 IP 地址。

- `port` (字符) - 请求的地址端口。

- `socket` (元组) - 请求的 (IP, port).

- `app` - 一个处理请求的 Sanic 程序对象的引用。当一些包含蓝图或其他处理函数的模块中没有访问 `app` 对象的权限时有用。

  ```python
  from sanic.response import json
  from sanic import Blueprint

  bp = Blueprint('my_blueprint')

  @bp.route('/')
  async def bp_root(request):
      if request.app.config['DEBUG']:
          return json({'status': 'debug'})
      else:
          return json({'status': 'production'})

  ```
- `url`: 请求的完整 URL， ie: `http://localhost:8000/posts/1/?foo=bar`
- `scheme`: 与请求相关联的 URL 方案: `http` 或 `https`
- `host`: 与请求相关联的主机: `localhost:8080`
- `path`: 请求的路径: `/posts/1/`
- `query_string`: 请求的参数: `foo=bar` 或者空的字符 `''`
- `uri_template`: 匹配路由程序的模板： `/posts/<id>/`
- `token`: 认证头部的值: `Basic YWRtaW46YWRtaW4=`


## 用 `get` 和 `getlist` 访问值

返回字典的请求数据实际返回了一个叫做 `RequestParameters` 的 `dict` 的子类。使用这个对象的关键区别在于 `get` 和 `getlist` 方法之间的区别。

- `get(key, default=None)` 正常执行，除非给定的 key 的值 是一个列表，*只返回第一个元素*。
- `getlist(key, default=None)` 正常执行，*返回整个列表*.

```python
from sanic.request import RequestParameters

args = RequestParameters()
args['titles'] = ['Post 1', 'Post 2']

args.get('titles') # => 'Post 1'

args.getlist('titles') # => ['Post 1', 'Post 2']
```
