# Python requests 库的高级应用 - 超时、重试、钩子（timeouts、retries、hooks）
_作者：[**Dani Hodovic**](https://hodovi.ch/)_  
_原文链接：<https://hodovi.ch/blog/advanced-usage-python-requests-timeouts-retries-hooks/>_  
_2020 年 2 月 28 日_  
`Crawler` `requests`

---
Python 的 requests 库可能是我在所有编程语言中最喜欢的 HTTP 处理库。
它简单、直观而且在 Python 社区中广泛传播。
大多数程序中，涉及 HTTP 接口的部分，基本都是使用 requests 或 urllib3 库来实现。

requests 库由于其简单的 API，很容易就可以应用实际生产中，不仅如此，该库还为高级用例提供了可扩展性。
如果你正在编写 API 密集型客户端或网络爬虫，你可能还需要处理网络故障、调试跟踪和语法糖之类的问题。 

下面的摘要是我在编写网络抓取工具或程序时，对于处理 JSON API 很有用的一些功能。
- [requests hooks](##请求钩子)
- Setting base URLs
- Setting default timeouts
- Retry on failure
  - Combining timeouts and retries
- Debugging HTTP headers
  - Printing HTTP headers
  - Printing everything
- Testing and mocking requests
- Mimicking browser behaviours




## 请求钩子
通常在使用第三方 API 时，你希望验证服务器的响应是否正常。
requests 提供了一个快捷的方法 raise_for_status() ，它判断响应 HTTP 状态代码不是 4xx 或 5xx，即请求没有导致客户端或服务器错误。

```python
response = requests.get('https://api.github.com/user/repos?page=1')
# 断言这个响应没有错误
response.raise_for_status()
```

有时你需要每个响应对象都调用 raise_for_status() ，这会导致大量重复代码。
幸运的是，requests 库提供了一个“钩子”接口，你可以在请求过程的某些部分附加回调。

我们可以使用钩子来确保每个响应对象都会调用 raise_for_status()。 

```python
# 创建自定义 requests 对象，修改全局模块抛出错误
http = requests.Session()

assert_status_hook = lambda response, *args, **kwargs: response.raise_for_status()
http.hooks["response"] = [assert_status_hook]

http.get("https://api.github.com/user/repos?page=1")

> HTTPError: 401 Client Error: Unauthorized for url: https://api.github.com/user/repos?page=1
```

<div align=center><img src="https://www.mlpowered.com/images/w2v_importance.png" width = '700'></div>
<div align=center><h6>Word2Vec：单词重要性</h6></div>

---
[返回目录](https://github.com/datugou/Article_Translation)
