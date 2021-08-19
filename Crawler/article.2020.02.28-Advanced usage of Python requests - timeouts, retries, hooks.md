# Python requests 库的高级应用 - 超时、重试、钩子（timeouts、retries、hooks）
_作者：[**Dani Hodovic**](https://hodovi.ch/)_  
_原文链接：<https://hodovi.ch/blog/advanced-usage-python-requests-timeouts-retries-hooks/>_  
_2020 年 2 月 28 日_  
`Crawler` `requests`

---
Python 的 [requests](https://requests.readthedocs.io/en/master/) 库可能是我在所有编程语言中最喜欢的 HTTP 处理库。
它简单、直观而且在 Python 社区中广泛传播。
大多数程序中，涉及 HTTP 接口的部分，基本都是使用 requests 或 urllib3 库来实现。

requests 库由于其简单的 API，很容易就可以应用实际生产中，不仅如此，该库还为高级用例提供了可扩展性。
如果你正在编写 API 密集型客户端或网络爬虫，你可能还需要处理网络故障、调试跟踪和语法糖之类的问题。 

下面的摘要是我在编写网络抓取工具或程序时，对于处理 JSON API 很有用的一些功能。
- [requests 钩子](#requests_hook)
- [设置默认主页的 URLs](#Setting_base_URLs)
- [设置默认超时](#Setting_default_timeouts)
- Retry on failure
  - Combining timeouts and retries
- Debugging HTTP headers
  - Printing HTTP headers
  - Printing everything
- Testing and mocking requests
- Mimicking browser behaviours




## <span id ='requests_hook'>requests 钩子</span>
通常在使用第三方 API 时，你希望验证服务器的响应是否正常。
requests 提供了一个快捷的方法 `raise_for_status()` ，它判断响应 HTTP 状态代码不是 4xx 或 5xx，即请求没有导致客户端或服务器错误。

```python
response = requests.get('https://api.github.com/user/repos?page=1')
# 断言这个响应没有错误
response.raise_for_status()
```

有时你需要每个响应对象都调用 `raise_for_status()`，这会导致大量重复代码。
幸运的是，requests 库提供了一个“钩子”接口，你可以在请求过程的某些部分附加回调。

我们可以使用钩子来确保每个响应对象都会调用 `raise_for_status()`。 

```python
# 创建自定义 requests 对象，修改全局模块抛出错误
http = requests.Session()

assert_status_hook = lambda response, *args, **kwargs: response.raise_for_status()
http.hooks["response"] = [assert_status_hook]

http.get("https://api.github.com/user/repos?page=1")

> HTTPError: 401 Client Error: Unauthorized for url: https://api.github.com/user/repos?page=1
```

## <span id='Setting_base_URLs'>设置默认主页的 URLs</span>
假设你只访问主页为 api.org 的 API。
代码中会为每个 http 调用重复协议和域： 

```python
requests.get('https://api.org/list/')
requests.get('https://api.org/list/3/item')
```

你可以使用 [BaseUrlSession](https://toolbelt.readthedocs.io/en/latest/sessions.html#baseurlsession) 来减少一些重复代码。
使 HTTP 请求指定默认的 url，这样在请求时只需指定资源路径即可。 

```python
from requests_toolbelt import sessions
http = sessions.BaseUrlSession(base_url="https://api.org")
http.get("/list")
http.get("/list/3/item")
```

NOTE：默认的 requests 库不包含 [requests toolbelt](https://github.com/requests/toolbelt)，所以在使用前需要安装。

## <span id='Setting_default_timeouts'>设置默认超时</span>
[requests 的官方文档](https://requests.readthedocs.io/en/master/user/quickstart/#timeouts)建议你在所有生产代码上设置超时。
如果你忘记设置超时，一个异常的服务器可能会导致你的应用程序挂起，尤其是在大多数 Python 代码是同步的情况下。

```python
requests.get('https://github.com/', timeout=0.001)
```

然而这包含了大量重复代码，如果有人忘了设置超时，并在生产中导致程序停止了，到时候你可能会气的掀桌子。

使用 [Transport Adapters](https://requests.readthedocs.io/en/master/user/advanced/#transport-adapters)，
可以为所有 HTTP 调用设置默认超时。
这确保即使开发人员忘记将 timeout=1 参数添加到他的个人调用中也能设置合理的超时，并且还允许你在单独调用的时候，覆盖掉默认的参数。

下面是具有默认超时的自定义 Transport Adapters 示例，其灵感来自[此 Github 评论](https://github.com/kennethreitz/requests/issues/3070#issuecomment-205070203)。 
我们在构造 http 客户端和 send() 方法时重写构造函数以提供默认超时，以确保在未提供超时参数时使用默认超时。 

```python
from requests.adapters import HTTPAdapter

DEFAULT_TIMEOUT = 5 # seconds

class TimeoutHTTPAdapter(HTTPAdapter):
    def __init__(self, *args, **kwargs):
        self.timeout = DEFAULT_TIMEOUT
        if "timeout" in kwargs:
            self.timeout = kwargs["timeout"]
            del kwargs["timeout"]
        super().__init__(*args, **kwargs)

    def send(self, request, **kwargs):
        timeout = kwargs.get("timeout")
        if timeout is None:
            kwargs["timeout"] = self.timeout
        return super().send(request, **kwargs)
```

```python
import requests

http = requests.Session()

# Mount it for both http and https usage
adapter = TimeoutHTTPAdapter(timeout=2.5)
http.mount("https://", adapter)
http.mount("http://", adapter)

# Use the default 2.5s timeout
response = http.get("https://api.twilio.com/")

# Override the timeout as usual for specific requests
response = http.get("https://api.twilio.com/", timeout=10)
```

<div align=center><img src="https://www.mlpowered.com/images/w2v_importance.png" width = '700'></div>
<div align=center><h6>Word2Vec：单词重要性</h6></div>

---
[返回目录](https://github.com/datugou/Article_Translation)
