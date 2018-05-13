# 配置

任何相当复杂的应用程序都需要配置，，这些配置不会被绑定到实际的代码中。对于不同环境和安装的设置可能也不同。

## 基础

Sanic 在应用程序对象的 `config` 属性中拥有配置。该配置对象仅是一个对象因此能被使用点号或者像一个字典一样修改:

```
app = Sanic('myapp')
app.config.DB_NAME = 'appdb'
app.config.DB_USER = 'appuser'
```

由于配置对象实际上是一个字典，你可以用它的 `update` 方法来一次性设置一些值:

```
db_settings = {
    'DB_HOST': 'localhost',
    'DB_NAME': 'appdb',
    'DB_USER': 'appuser'
}
app.config.update(db_settings)
```

一般来说，惯例只能有大写的配置参数。下面描述的用于加载配置的方法仅查找这样的大写参数。

## 加载配置

下面是一些如果加载配置的方法。

### 从环境变量

任何被 `SANIC_` 前缀定义的变量将会被 sanic 配置接受。例如，设置 `SANIC_REQUEST_TIMEOUT` 将被应用程序自动加载并提供给 `REQUEST_TIMEOUT` 配置变量。你可以传递一个不同的前缀给 Sanic:

```python
app = Sanic(load_env='MYAPP_')
```

以上的变量会是 `MYAPP_REQUEST_TIMEOUT`。如果你想要禁用从环境变量加载你可以用 `False` 替代它的设置:

```python
app = Sanic(load_env=False)
```

### 从一个对象

如果有很多配置值并且它们有合理的默认值，将它们放入模块可能会有所帮助：

```
import myapp.default_settings

app = Sanic('myapp')
app.config.from_object(myapp.default_settings)
```

你也可以用一个类或者其他任何对象。

### 从一个文件

通常你想要从一个不是分布式应用程序部分的文件加载配置。你可以使用 `from_pyfile(/path/to/config_file)` 来加载。但是，这需要程序知道配置文件的路径。相反，你可以在环境变量中指定配置文件的位置，并告诉 Sanic 使用它来查找配置文件。

```
app = Sanic('myapp')
app.config.from_envvar('MYAPP_SETTINGS')
```

然后，你可以使用 `MYAPP_SETTINGS` 环境变量集运行您的应用程序：

```
$ MYAPP_SETTINGS=/path/to/config_file python3 myapp.py
INFO: Goin' Fast @ http://0.0.0.0:8000
```

配置文件是为了加载它们而执行的普通 Python 文件。这允许你使用任意逻辑来建立正确的配置。只有大写的变量才能加入配置。最常见的配置由简单的键值对组成：

```
# config_file
DB_HOST = 'localhost'
DB_NAME = 'appdb'
DB_USER = 'appuser'
```

## 内建的配置值

开箱即用只有几个预定的能在创建应用程序时被复写的值。

    | Variable           | Default   | Description                                   |
    | ------------------ | --------- | --------------------------------------------- |
    | REQUEST_MAX_SIZE   | 100000000 | 请求最大值 (bytes)              |
    | REQUEST_TIMEOUT    | 60        | 请求到达时间 (sec)   |
    | RESPONSE_TIMEOUT   | 60        | 响应处理时间 (sec) |
    | KEEP_ALIVE         | True      | 当 False 时禁用 keep-alive                |
    | KEEP_ALIVE_TIMEOUT | 5         | 一个 TCP 连接保持的时长 (sec)  |

### 不同的超时变量:

一个请求超时衡量了在一个新的开放的 TCP 连接传递到 Sanic 后端服务的瞬间，和当整个 HTTP 请求已经接收的瞬间之间的时间差。如果这个时间超过了 `REQUEST_TIMEOUT` 值 (秒)，这被认为是客户端错误，因此 Sanic 生成一个 HTTP 408 响应并发送给客户端。调高这个值如果你的客户端经常传递非常大的请求负荷或者上传请求非常慢。

一个响应超时衡量了在一个 Sanic 服务器传递 HTTP 请求到 Sanic APP的瞬间，和一个 HTTP 响应发送到客户端的瞬间之间的时间差。如果这个时间超过了 `RESPONSE_TIMEOUT` 值 (秒)，这被认为是服务器的错误，因此 Sanic 生成一个 HTTP 503 响应并设置响应给客户端。调高这个值如果你的应用程序很可能有长时间运行的程序导致延迟了响应的产生。

### 什么是 Keep Alive？Keep Alive Timeout 是干嘛的？

Keep-Alive 是一个在 HTTP 1.1 里介绍的 HTTP 功能。当发送一个 HTTP 请求时，这个客户端 (通常是一个网络浏览器) 可以设置一个 Keep-Alive 头来指示 http 服务器 (Sanic) 不要在发送响应后关闭 TCP 连接。这允许客户端复用存在的 TCP 连接来发送随后的 HTTP 请求，确保为客户端和服务端提供更加高效的网络质量。

`KEEP_ALIVE` 配置变量默认被设置为 `True`。如果你的应用程序不需要这个功能，设置 `False` 使所有客户端连接在响应被发送后立即关闭，无论请求的 Keep-Alive 头如何。

服务器保持打开 TCP 连接的时间量由服务器自己决定。在 Sanic，这个值通过 `KEEP_ALIVE_TIMEOUT` 值来配置。默认是 5 秒，与 Apache HTTP 服务器一样的配置，这在允许足够的时间为客户端发送一个新的请求和不用一次性保持太多连接之间提供了很好的平衡。不要超过 75 秒除非你知道你的客户端使用了支持 TCP 长连接。

参考：
```
Apache httpd server default keepalive timeout = 5 seconds
Nginx server default keepalive timeout = 75 seconds
Nginx performance tuning guidelines uses keepalive = 15 seconds
IE (5-9) client hard keepalive limit = 60 seconds
Firefox client hard keepalive limit = 115 seconds
Opera 11 client hard keepalive limit = 120 seconds
Chrome 13+ client keepalive limit > 300+ seconds
```
