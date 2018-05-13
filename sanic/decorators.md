# 处理装饰器

由于 Sanic 处理程序是简单的 Python 函数，你可以用类似 Flask 的方法用装饰器包裹他们。一个经典的用例就是当你想要在处理程序的代码执行前先运行一些代码。

## 认证装饰器

假设你想要检查一个用户是否被授权去访问特定端点。你可以创建一个装饰器来包裹一个处理程序，检查一个请求是否客户端被授权去访问一个资源，并且发送一个合理的响应。


```python
from functools import wraps
from sanic.response import json

def authorized():
    def decorator(f):
        @wraps(f)
        async def decorated_function(request, *args, **kwargs):
            # run some method that checks the request
            # for the client's authorization status
            is_authorized = check_request_for_authorization_status(request)

            if is_authorized:
                # the user is authorized.
                # run the handler method and return the response
                response = await f(request, *args, **kwargs)
                return response
            else:
                # the user is not authorized.
                return json({'status': 'not_authorized'}, 403)
        return decorated_function
    return decorator


@app.route("/")
@authorized()
async def test(request):
    return json({status: 'authorized'})
```
