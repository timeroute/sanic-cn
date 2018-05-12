# 开始

在开始前请确保你同时安装了  [pip](https://pip.pypa.io/en/stable/installing/) 和至少 3.5 以上版本的 Python 。Sanic 使用了新的 `async`/`await`
语法，所以之前的版本无法工作。

1. 安装 Sanic: `python3 -m pip install sanic`
2. 新建一个叫 `main.py` 的文件并且附上以下代码：

  ```python
  from sanic import Sanic
  from sanic.response import json

  app = Sanic()

  @app.route("/")
  async def test(request):
      return json({"hello": "world"})

  if __name__ == "__main__":
      app.run(host="0.0.0.0", port=8000)
  ```

3. 启动服务器: `python3 main.py`
4. 在你的浏览器打开地址 `http://0.0.0.0:8000`。你会看到 *Hello world!*。

现在你已经会用 Sanic 了！
