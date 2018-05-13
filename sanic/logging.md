# 日志


Sanic 允许你在基于 [python3 logging API](https://docs.python.org/3/howto/logging.html) 的请求上做不同类型的日志 (access log, error log)。如果你想创建一个新的配置，你应该了解 python3 日志模块的基本知识。

### 快速开始

一个使用默认设置的简单例子如下：

```python
from sanic import Sanic

app = Sanic('test')

@app.route('/')
async def test(request):
    return response.text('Hello World!')

if __name__ == "__main__":
  app.run(debug=True, access_log=True)
```

要使用你自己的日志配置，简单使用 `logging.config.dictConfig`，或者在你初始化 `Sanic` 应用的时候传递 `log_config`：

```python
app = Sanic('test', log_config=LOGGING_CONFIG)
```

要关闭日志，只需指定 access_log=False:

```python
if __name__ == "__main__":
  app.run(access_log=False)
```

这会在处理请求时跳过调用日志记录程序。并且你甚至可以在生产中进一步提高速度：

```python
if __name__ == "__main__":
  # disable debug messages
  app.run(debug=False, access_log=False)
```

### 配置

默认情况下，log_config 参数设置为使用 sanic.log.LOGGING_CONFIG_DEFAULTS 字典进行配置。

sanic 使用了三种 `loggers`，并且 **当你想要创建你自己的日志配置时必须被定义**:

- root:<br>
  用来记录内部消息。

- sanic.error:<br>
  用来记录错误日志。

- sanic.access:<br>
  用来记录访问日志。

#### 日志格式化：

除了 python 提供的默认参数 (asctime, levelname, message) 外，Sanic 提供了额外的参数给访问日志：

- host (str)<br>
  request.ip


- request (str)<br>
  request.method + " " + request.url


- status (int)<br>
  response.status


- byte (int)<br>
  len(response.body)


默认的访问日志格式如下：

```python
%(asctime)s - (%(name)s)[%(levelname)s][%(host)s]: %(request)s %(message)s %(status)d %(byte)d
```
