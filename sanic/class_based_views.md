# 基于类的视图

基于类的视图只是实现对请求的响应行为的类。它们提供了在同一个端点上划分不同HTTP请求类型的处理方式。相当与定义和装饰了三种不同的处理程序，每端支持一种请求类型，端点可以被分配一个基于类的视图。

## 定义视图

一个基于类的视图应该继承 `HTTPMethodView`。你可以为每个你想要支持的 HTTP 请求类型执行类方法。如果请求已经接收但是没有定义方法，一个 `405: Method not allowed` 响应就会生成。

要注册一个基于类的视图到端，`app.add_route` 方法是有用的。第一个参数应该是被定义带 `as_view` 方法的类，第二个参数应该是 URL 端。

有效的方法有 `get`, `post`, `put`, `patch` 和 `delete`。一个使用所有方法的类看起来如下。

```python
from sanic import Sanic
from sanic.views import HTTPMethodView
from sanic.response import text

app = Sanic('some_name')

class SimpleView(HTTPMethodView):

  def get(self, request):
      return text('I am get method')

  def post(self, request):
      return text('I am post method')

  def put(self, request):
      return text('I am put method')

  def patch(self, request):
      return text('I am patch method')

  def delete(self, request):
      return text('I am delete method')

app.add_route(SimpleView.as_view(), '/')

```

你也可以使用 `async` 语法。

```python
from sanic import Sanic
from sanic.views import HTTPMethodView
from sanic.response import text

app = Sanic('some_name')

class SimpleAsyncView(HTTPMethodView):

  async def get(self, request):
      return text('I am async get method')

app.add_route(SimpleAsyncView.as_view(), '/')

```

## URL 参数

如果需要路由指南中讨论的任何URL参数，请将其包括在方法定义中。

```python
class NameView(HTTPMethodView):

  def get(self, request, name):
    return text('Hello {}'.format(name))

app.add_route(NameView.as_view(), '/<name>')
```

## 装饰器

如果你想要添加任何装饰器到类，你可以设置 `decorators` 类的变量。这些会在 `as_view` 被调用时应用到类。

```python
class ViewWithDecorator(HTTPMethodView):
  decorators = [some_decorator_here]

  def get(self, request, name):
    return text('Hello I have a decorator')

  def post(self, request, name):
    return text("Hello I also have a decorator")

app.add_route(ViewWithDecorator.as_view(), '/url')
```

但是如果你只想装饰一些函数而不是所有，你可以操作如下：

```python
class ViewWithSomeDecorator(HTTPMethodView):

    @staticmethod
    @some_decorator_here
    def get(request, name):
        return text("Hello I have a decorator")

    def post(self, request, name):
        return text("Hello I don't have any decorators")
```

## URL 构建

如果你希望为 HTTPMethodView 构建 URL，记住类名将成为你传入 `url_for` 的端点。例如：

```python
@app.route('/')
def index(request):
    url = app.url_for('SpecialClassView')
    return redirect(url)


class SpecialClassView(HTTPMethodView):
    def get(self, request):
        return text('Hello from the Special Class View!')


app.add_route(SpecialClassView.as_view(), '/special_class_view')
```


## 使用 CompositionView

作为 `HTTPMethodView` 的替代方案，你可用 `CompositionView` 在视图类之外移动处理函数。

每个支持的 HTTP 方法的处理程序都在源文件的其他地方定义了，然后使用 `CompositionView.add` 方法添加到视图。第一个参数是一个要处理的 HTTP 方法列表 (e.g. `['GET', 'POST']`)，第二个是处理程序。下面的例子展示了带两个外部处理程序和一个内部匿名函数的 `CompositionView` 的用法：

```python
from sanic import Sanic
from sanic.views import CompositionView
from sanic.response import text

app = Sanic(__name__)

def get_handler(request):
    return text('I am a get method')

view = CompositionView()
view.add(['GET'], get_handler)
view.add(['POST', 'PUT'], lambda request: text('I am a post/put method'))

# Use the new view to handle requests to the base URL
app.add_route(view, '/')
```

注意：当前你不能通过 `url_for` 为 CompositionView 建立 URL。
