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
- [访问失败后重试](#Retry_on_failure)
  - [超时和重试结合使用](#Combining_timeouts_and_retries)
- [调试 HTTP 请求](#Debugging_HTTP_headers)
  - [打印 HTTP headers](#Printing_HTTP_headers)
  - [打印所有信息](#Printing_everything)
- [测试和模拟请求 ](#Testing_and_mocking_requests)
- [模仿浏览器的行为](#Mimicking_browser_behaviours)




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

<div align=center><img src="https://media.giphy.com/media/6IZzimIQN8guI/giphy.gif" width = '700'></div>


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


## <span id='Retry_on_failure'>访问失败后重试</span>
网络连接有时会出现丢失、拥塞和服务器故障等情况。
如果我们想构建一个真正强大稳定的程序，我们需要考虑容纳这些失败的访问并制定重试策略。

在 HTTP 客户端添加重试策略非常简单。
创建一个 HTTPAdapter 并将我们的策略传递给它。

```python
from requests.adapters import HTTPAdapter
from requests.packages.urllib3.util.retry import Retry

retry_strategy = Retry(
    total=3,
    status_forcelist=[429, 500, 502, 503, 504],
    method_whitelist=["HEAD", "GET", "OPTIONS"]
)
adapter = HTTPAdapter(max_retries=retry_strategy)
http = requests.Session()
http.mount("https://", adapter)
http.mount("http://", adapter)

response = http.get("https://en.wikipedia.org/w/api.php")
```

初始的 Retry 类会提供合理的默认值，也可以自己配置，这里是我最常用的参数。

下面的参数包括 requests 库使用的默认参数。 

```python
total = 10
```

重试尝试的总数。
如果失败的请求或重定向的数量超过此数量，客户端将抛出 `urllib3.exceptions.MaxRetryError` 异常。
根据所使用 API 的稳定情况改变这个参数，但我通常将它设置为低于 10，一半 3 次重试就足够了。

```python
status_forcelist=[413, 429, 503]
```

如果服务器返回这些 HTTP 响应码就重试 。
你可能希望重试常见的服务器错误（500、502、503、504），因为服务器和反向代理并不总是遵守 HTTP 规范。
重试状态码最好始终包含 429 (访问频率超过限制) 状态码，因为 urllib 库在默认情况下应该对失败的请求进行递增，导致短时间内访问服务器次数过多。 

```python
method_whitelist=["HEAD", "GET", "PUT", "DELETE", "OPTIONS", "TRACE"]
```

仅在这些 HTTP 方法失败时重试。
默认情况下，这包括除 POST 之外的所有 HTTP 方法，因为 POST 可能会导致新的插入。
修改此参数以包含 POST，因为我处理的大多数 API 不会返回错误代码并在同一个调用中执行插入。
如果服务器确实这样做了，你应该向他们提交错误报告。 

```python
backoff_factor=0
```

这是一个有趣的参数。
它允许你更改进程在失败请求之间等待的时间。
算法如下： 

```python
{backoff factor} * (2 ** ({number of total retries} - 1))
```

例如，如果 backoff_factor 设置为：

- 1 秒 - 重试间隔的时间为 `0.5、1、2、4、8、16、32、64、128、256`
- 2 秒 - `1、2、4、8、16、32、64、128、256、512`
- 10 秒 - `5、10、20、40、80、160、320、640、1280、2560`

该值呈指数增长，这是比较合理的[重试策略](https://stackoverflow.com/a/28732630/2966951)实现方式。
通过设置 backoff_factor 参数，我们可以决定每次睡眠乘以多少。

该值默认为 0，这意味着重试之间不会有间隔时间，重试将立即执行。
__确保将此设置为 1 以避免对服务器造成压力！__

关于重试模块的完整文档在[这里](https://urllib3.readthedocs.io/en/latest/reference/urllib3.util.html#module-urllib3.util.retry)。 

### <span id='Combining_timeouts_and_retries'>超时和重试结合使用</span>

由于 HTTPAdapter 对象是类似的，我们可以像这样把重试和超时结合起来：

```python
retries = Retry(total=3, backoff_factor=1, status_forcelist=[429, 500, 502, 503, 504])
http.mount("https://", TimeoutHTTPAdapter(max_retries=retries))
```


## <span id = 'Debugging_HTTP_requests'>调试 HTTP 请求</span>

有时请求失败，你想弄清楚原因。
记录请求和响应可能会让你深入了解访问为何失败。
有两种方法可以做到这一点 - 使用内置的调试日志设置或使用请求钩子。 

### <span id = 'Printing_HTTP_headers'>打印 HTTP headers</span>
将日志调试级别更改为大于 0 将记录响应 HTTP headers。
这是最简单的选项，但它不允许您查看 HTTP 请求或响应的正文。
如果你处理的 API 响应返回的正文太大或包含二进制字符，日记记录这些内容显然不合适，这种情况下，只打印 HTTP headers 将非常合适。

任何大于 0 的值都将启用调试日志记录。 

```python
import requests
import http

http.client.HTTPConnection.debuglevel = 1

requests.get("https://www.google.com/")

# Output
send: b'GET / HTTP/1.1\r\nHost: www.google.com\r\nUser-Agent: python-requests/2.22.0\r\nAccept-Encoding: gzip, deflate\r\nAccept: */*\r\nConnection: keep-alive\r\n\r\n'
reply: 'HTTP/1.1 200 OK\r\n'
header: Date: Fri, 28 Feb 2020 12:13:26 GMT
header: Expires: -1
header: Cache-Control: private, max-age=0
```

### <span id = 'Printing_everything'>打印所有信息</span>
  
如果你想记录整个 HTTP 生命周期，包括请求和响应的文本内容，你可以使用请求钩子和来自 requests_toolbelt 库的转储工具。

每当我处理不返回非常大的响应的基于 REST 的 API 时，我更喜欢用这种方式。 

```python
import requests
from requests_toolbelt.utils import dump

def logging_hook(response, *args, **kwargs):
    data = dump.dump_all(response)
    print(data.decode('utf-8'))

http = requests.Session()
http.hooks["response"] = [logging_hook]

http.get("https://api.openaq.org/v1/cities", params={"country": "BA"})

# Output
< GET /v1/cities?country=BA HTTP/1.1
< Host: api.openaq.org

> HTTP/1.1 200 OK
> Content-Type: application/json; charset=utf-8
> Transfer-Encoding: chunked
> Connection: keep-alive
>
{
   "meta":{
      "name":"openaq-api",
      "license":"CC BY 4.0",
      "website":"https://docs.openaq.org/",
      "page":1,
      "limit":100,
      "found":1
   },
   "results":[
      {
         "country":"BA",
         "name":"Goražde",
         "city":"Goražde",
         "count":70797,
         "locations":1
      }
   ]
}
```

[参见](https://toolbelt.readthedocs.io/en/latest/dumputils.html)


## <span id='Testing_and_mocking_requests'>测试和模拟请求 </span>

使用第三方 API 会给开发带来一个痛点：它们很难进行单元测试。
[Sentry](https://sentry.io/welcome/) 的工程师编写了一个库来模拟开发过程中的请求，从而缓解了这种痛苦。

[getentry/responses](https://github.com/getsentry/responses) 拦截了 HTTP 请求并返回你在测试期间添加的预定义响应，
而不是将 HTTP 响应发送到服务器。

最好用一个例子来演示。 


```python
import unittest
import requests
import responses


class TestAPI(unittest.TestCase):
    @responses.activate  # intercept HTTP calls within this method
    def test_simple(self):
        response_data = {
                "id": "ch_1GH8so2eZvKYlo2CSMeAfRqt",
                "object": "charge",
                "customer": {"id": "cu_1GGwoc2eZvKYlo2CL2m31GRn", "object": "customer"},
            }
        # mock the Stripe API
        responses.add(
            responses.GET,
            "https://api.stripe.com/v1/charges",
            json=response_data,
        )

        response = requests.get("https://api.stripe.com/v1/charges")
        self.assertEqual(response.json(), response_data)
```

如果发出与模拟响应不匹配的 HTTP 请求，则会引发 ConnectionError。

```python
class TestAPI(unittest.TestCase):
    @responses.activate
    def test_simple(self):
        responses.add(responses.GET, "https://api.stripe.com/v1/charges")
        response = requests.get("https://invalid-request.com")
```

输出结果

```python
requests.exceptions.ConnectionError: Connection refused by Responses - the call doesn't match any registered mock.

Request:
- GET https://invalid-request.com/

Available matches:
- GET https://api.stripe.com/v1/charges
```

## <span id='Mimicking_browser_behaviours'>模仿浏览器的行为</span>

如果你编写了足够多的网络爬虫代码，你会注意到某些网站如何返回不同的 HTML，具体取决于你是使用浏览器还是以编程方式访问网站。
有时这是一种反抓取措施，但通常服务器会参与 User-Agent 嗅探，以找出最适合设备（例如桌面或移动设备）的内容。

如果你想返回与浏览器显示相同的内容，可以使用 Firefox 或 Chrome 的 User-Agent 来覆盖 request headers 的默认值。 

```python
import requests
http = requests.Session()
http.headers.update({
    "User-Agent": "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:68.0) Gecko/20100101 Firefox/68.0"
})
```


---
[返回目录](https://github.com/datugou/Article_Translation)
