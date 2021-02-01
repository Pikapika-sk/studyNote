# urllib

## 1.urllib.request.urlopen

~~~ python
import urllib.request

response = urllib.request.urlopen('https://www.python.org')
# print response.read().decode('utf-8'))
print(type(response))
print(response.status)
print(response.getheaders)
print(response.getheader('Server'))

# output:
# <class 'http.client.HTTPResponse'>
# 200
# <bound method HTTPResponse.getheaders of <http.client.HTTPResponse object at 0x000001E773FEA970>>
# nginx
~~~

函数的API

~~~ python
urllib.request.urlopen(url,data= None ,[timeout,]*, cafile= =None ,capath =None ,cadefault = Fault , context = None)
~~~

## 2.  urllib.request.Request

~~~ python
lass urllib.request.Request(url, data=None, headers={}, origin_req_host=None, unverifiable=False, method=None)
~~~

- 第一个参数 url 用于请求 URL，这是必传参数，其他都是可选参数。

- 第二个参数 data 如果要传，必须传 bytes（字节流）类型的。如果它是字典，可以先用 urllib.parse 模块里的 urlencode() 编码。

- 第三个参数 headers 是一个字典，它就是请求头，我们可以在构造请求时通过 headers 参数直接构造，也可以通过调用请求实例的 add_header() 方法添加。添加请求头最常用的用法就是通过修改 `User-Agent` 来伪装浏览器，默认的 User-Agent 是 Python-urllib，我们可以通过修改它来伪装浏览器。比如要伪装火狐浏览器，你可以把它设置为：

~~~ python
Mozilla/5.0 (X11; U; Linux i686) Gecko/20071127 Firefox/2.0.0.11
~~~

- 第四个参数 origin_req_host 指的是请求方的 `host 名称`或者 `IP 地址`。
- 第五个参数 unverifiable 表示这个请求`是否是无法验证`的，默认是 False，意思就是说用户没有足够权限来选择接收这个请求的结果。例如，我们请求一个 HTML 文档中的图片，但是我们没有自动抓取图像的权限，这时 unverifiable 的值就是 True。
- 第六个参数 method 是一个字符串，用来指示请求使用的方法，比如 GET、POST 和 PUT 等。

~~~ python
from urllib import request, parse  

url = 'http://httpbin.org/post'  
headers = {'User-Agent': 'Mozilla/4.0 (compatible; MSIE 5.5; Windows NT)',  
    'Host': 'httpbin.org'  
}  
dict = {'name': 'Germey'}  
data = bytes(parse.urlencode(dict), encoding='utf8')  
req = request.Request(url=url, data=data, headers=headers, method='POST')  
response = request.urlopen(req)  
print(response.read().decode('utf-8'))
~~~

headers可以用 `add_header()`方法来添加

```python
req = request.Request(url=url, data=data, method='POST')  
req.add_header('User-Agent', 'Mozilla/4.0 (compatible; MSIE 5.5; Windows NT)')
```

## 3.Handler

- 专门处理登录验证、cookies，代理的

	首先，介绍一下 urllib.request 模块里的 BaseHandler 类，它是所有其他 Handler 的父类，它提供了最基本的方法，例如 default_open、protocol_request 等。

	接下来，就有各种 Handler 子类继承这个 BaseHandler 类，举例如下。

	- HTTPDefaultErrorHandler 用于处理 HTTP 响应错误，错误都会抛出 HTTPError 类型的异常。
	- HTTPRedirectHandler 用于处理重定向。
	- HTTPCookieProcessor 用于处理 Cookies。
	- ProxyHandler 用于设置代理，默认代理为空。
	- HTTPPasswordMgr 用于管理密码，它维护了用户名密码的表。
	- HTTPBasicAuthHandler 用于管理认证，如果一个链接打开时需要认证，那么可以用它来解决认证问题。
	- urlopen 这个方法，实际上它就是 urllib 为我们提供的一个 Opener。
	- 简而言之 **用`handler`来构建`Opener**`

	

#### a.验证页面

```python
from urllib.request import HTTPPasswordMgrWithDefaultRealm, HTTPBasicAuthHandler, build_opener  
from urllib.error import URLError  

username = 'username'  
password = 'password'  
url = 'http://localhost:5000/'  

p = HTTPPasswordMgrWithDefaultRealm()  
p.add_password(None, url, username, password)  # 添加用户名密码
auth_handler = HTTPBasicAuthHandler(p)     # 实例化HTTPBasicAuthHandler对象
opener = build_opener(auth_handler)  

try:  
    result = opener.open(url)  
    html = result.read().decode('utf-8')  
    print(html)  
except URLError as e:  
    print(e.reason)		
```

### b.使用代理

```python
from urllib.error import URLError  
from urllib.request import ProxyHandler, build_opener  

proxy_handler = ProxyHandler({  
    'http': 'http://127.0.0.1:9743',  
    'https': 'https://127.0.0.1:9743'  
})  
opener = build_opener(proxy_handler)  
try:  
    response = opener.open('https://www.baidu.com')  
    print(response.read().decode('utf-8'))  
except URLError as e:  
    print(e.reason)
```

### c.Cookies处理

将网站的 Cookies 获取下来

```python
import http.cookiejar
import urllib.request

cookie = http.cookiejar.CookieJar()
handler = urllib.request.HTTPCookieProcessor(cookie)
opener = urllib.request.build_opener(handler)
response = opener.open('https://www.baidu.com')
for item in cookie:
    print(item.name+" = " + item.value)

```

以文件格式输出

```python
import http.cookiejar
import urllib.request

filename = 'cookies.txt'
cookie = http.cookiejar.MozillaCookieJar(filename)
handler = urllib.request.HTTPCookieProcessor(cookie)
opener = urllib.request.build_opener(handler)
response = opener.open('http://www.baidu.com')
cookie.save(ignore_discard=True, ignore_expires=True)
```

读取Cookie文件并利用

```python
import http.cookiejar
import urllib.request

cookie = http.cookiejar.LWPCookieJar()
cookie.load('cookies.txt', ignore_discard=True, ignore_expires=True)
handler = urllib.request.HTTPCookieProcessor(cookie)
opener = urllib.request.build_opener(handler)
response = opener.open('http://www.baidu.com')
print(response.read().decode('utf-8'))
```

可以看到，这里调用 load 方法来读取本地的 Cookies 文件，获取到了 Cookies 的内容。不过前提是我们首先生成了 LWPCookieJar 格式的 Cookies，并保存成文件，然后读取 Cookies 之后使用同样的方法构建 Handler 和 Opener 即可完成操作。

## 4.处理异常

### URLError

```python
from urllib import request, error
try:
    respoense = request.urlopen('https://cuiqingcai.com/index.hmt')
except error.URLError as e:
    print(e.reason)
```

- URLError 类来自 urllib 库的 error 模块，它继承自 OSError 类，是 error 异常模块的基类，由 request 模块产生的异常都可以通过捕获这个类来处理。
- 它具有一个属性 reason，即返回错误的原因。

### HTTPError

它是 URLError 的子类，专门用来处理 HTTP 请求错误，比如认证请求失败等。它有如下 3 个属性。

- code：返回 HTTP 状态码，比如 404 表示网页不存在，500 表示服务器内部错误等。
- reason：同父类一样，用于返回错误的原因。
- headers：返回请求头。

### URLError是HTTPError的父类

```python
# 更好的写法
from urllib import request, error  

try:  
    response = request.urlopen('https://cuiqingcai.com/index.htm')  
except error.HTTPError as e:  # 在这里就异常就被捕捉了
    print(e.reason, e.code, e.headers, sep='\n')  
except error.URLError as e:  
    print(e.reason)  
else:  
    print('Request Successfully')
```

## 5.解析链接

#### fragment

　　主要资源是由 URI 进行标识，URI 中的 fragment 用来标识次级资源。我理解看来，fragment 主要是用来标识 URI 所标识资源里的某个资源。

　　在 URI 的末尾通过 hash mark（#）作为 fragment 的开头，其中 # 不属于 fragment 的值。

　　`https://domain/index#L18`

　　这个 URI 中 `L18` 就是 fragment 的值。这有哪些特殊的地方呢？

1. `#` 有别于 `?`，`?` 后面的查询字符串会被网络请求带上服务器，而 fragment 不会被发送的服务器；
2. fragment 的改变不会触发浏览器刷新页面，但是会生成浏览历史；
3. fragment 会被浏览器根据文件媒体类型（MIME type）进行对应的处理；
4. Google 的搜索引擎会忽略 `#` 及其后面的字符串。

#### urlparse

- 实现URL的识别和分段
- 出一个标准的链接格式，具体如下：

> scheme://netloc/path;params?query#fragment

一个标准的 URL 都会符合这个规则，利用 urlparse 方法可以将它拆分开来。

- allow_fragments：即是否忽略 fragment。如果它被设置为 False，fragment 部分就会被忽略，它会被解析为 path、parameters 或者 query 的一部分，而 fragment 部分为空。
- 当 URL 中不包含 params 和 query 时，fragment 便会被解析为 path 的一部分。

```python
urllib.parse.urlparse(urlstring, scheme='', allow_fragments=True)
```

```python
from urllib.parse import urlparse

result = urlparse('http://www.baidu.com/index.html;uesr?id=5#comment')
print(type(result), result)

#output
# <class 'urllib.parse.ParseResult'> ParseResult(scheme='http', netloc='www.baidu.com', path='/index.html', params='uesr', query='id=5', fragment='comment')

```

| scheme   | 0    | URL 协议               | scheme 参数 |
| -------- | ---- | ---------------------- | ----------- |
| netloc   | 1    | 域名及网络端口         | 空字符串    |
| path     | 2    | 分层路径               | 空字符串    |
| params   | 3    | 最后一个路径元素参数   | 空字符串    |
| query    | 4    | Query 组件             | 空字符串    |
| fragment | 5    | 片段标志符             | 空字符串    |
| username |      | 用户名                 | None        |
| password |      | Password               | None        |
| hostname |      | 主机名 (小写)          | None        |
| port     |      | 如果存在，端口值为整数 | None        |

#### urlunparse

- 进行url的构造

- 它接受的参数是一个可迭代对象，但是它的长度必须是 **6**，否则会抛出参数数量不足或者过多的问题

```python
from urllib.parse import urlunparse  

data = ['http', 'www.baidu.com', 'index.html', 'user', 'a=6', 'comment']  
print(urlunparse(data))
```

#### urlsplit

- 这个方法和 urlparse 方法非常相似，只不过它不再单独解析 params 这一部分，只返回 5 个结果。上面例子中的 params 会合并到 path 中

```python
from urllib.parse import urlsplit  

result = urlsplit('http://www.baidu.com/index.html;user?id=5#comment')  
print(result)

#output
#SplitResult(scheme='http', netloc='www.baidu.com', path='/index.html;user', query='id=5', fragment='comment')
```

#### urlunsplit

- urlunparse 方法类似，它也是将链接各个部分组合成完整链接的方法，传入的参数也是一个可迭代对象，例如列表、元组等，唯一的区别是长度必须为 5

```python
from urllib.parse import urlunsplit  

data = ['http', 'www.baidu.com', 'index.html', 'a=6', 'comment']  
print(urlunsplit(data))

#http://www.baidu.com/index.html?a=6#comment
```

#### urljoin

生成链接还有另一个方法，那就是 urljoin 方法。我们可以提供一个 base_url（基础链接）作为第一个参数，将新的链接作为第二个参数，该方法会分析 base_url 的 scheme、netloc 和 path 这 3 个内容并对新链接缺失的部分进行补充，最后返回结果。

####  urlencode

- 在构造 GET 请求参数的时候非常有用

```python
from urllib.parse import urlencode  

params = {  
    'name': 'germey',  
    'age': 22  
}  
base_url = 'http://www.baidu.com?'  
url = base_url + urlencode(params)  
print(url)
```

这里首先声明了一个字典来将参数表示出来，然后调用 urlencode 方法将其序列化为 GET 请求参数。

运行结果如下：

```python
http://www.baidu.com?name=germey&amp;age=22
```

可以看到，参数就成功地由字典类型转化为 GET 请求参数了。

这个方法非常常用。有时为了更加方便地构造参数，我们会事先用字典来表示。要转化为 URL 的参数时，只需要调用该方法即可。

#### parse_qs

反序列化。如果我们有一串 GET 请求参数，利用 parse_qs 方法，就可以将它转回字典，

```python
from urllib.parse import parse_qs  

query = 'name=germey&amp;age=22'  
print(parse_qs(query))
```

运行结果如下：

```python
{'name': ['germey'], 'age': ['22']}
```

#### parse_qsl

它用于将参数转化为元组组成的列表

```python
from urllib.parse import parse_qsl  

query = 'name=germey&amp;age=22'  
print(parse_qsl(query))
```

运行结果如下：

```python
[('name', 'germey'), ('age', '22')]
```

#### quote

该方法可以将内容转化为 URL 编码的格式。URL 中带有中文参数时，有时可能会导致乱码的问题，此时用这个方法可以将中文字符转化为 URL 编码

```python
from urllib.parse import quote  

keyword = ' 壁纸 '  
url = 'https://www.baidu.com/s?wd=' + quote(keyword)  
print(url)

#https://www.baidu.com/s?wd=% E5% A3%81% E7% BA% B8
```

####  unquote

它可以进行 URL 解码

```python
from urllib.parse import unquote  

url = 'https://www.baidu.com/s?wd=% E5% A3%81% E7% BA% B8'  
print(unquote(url))
```

这是上面得到的 URL 编码后的结果，这里利用 unquote 方法还原，结果如下：

```python
https://www.baidu.com/s?wd = 壁纸
```

### robotparser

禁止所有爬虫访问任何目录的代码如下：

```
User-agent: *   
Disallow: /
```

允许所有爬虫访问任何目录的代码如下：

```
User-agent: *  
Disallow:
```

robotparser 模块，我们可以实现网站 Robots 协议的分析

```python
urllib.robotparser.RobotFileParser(url='')
```

这个类常用的几个方法

- set_url ：用来设置 robots.txt 文件的链接。如果在创建 RobotFileParser 对象时传入了链接，那么就不需要再使用这个方法设置了。
- read：读取 robots.txt 文件并进行分析。注意，这个方法执行一个读取和分析操作，如果不调用这个方法，接下来的判断都会为 False，所以一定记得调用这个方法。这个方法不会返回任何内容，但是执行了读取操作。
- parse：用来解析 robots.txt 文件，传入的参数是 robots.txt 某些行的内容，它会按照 robots.txt 的语法规则来分析这些内容。
- can_fetch：该方法传入两个参数，第一个是 User-agent，第二个是要抓取的 URL。返回的内容是该搜索引擎是否可以抓取这个 URL，返回结果是 True 或 False。
- mtime：返回的是上次抓取和分析 robots.txt 的时间，这对于长时间分析和抓取的搜索爬虫是很有必要的，你可能需要定期检查来抓取最新的 robots.txt。
- modified：它同样对长时间分析和抓取的搜索爬虫很有帮助，将当前时间设置为上次抓取和分析 robots.txt 的时间。

~~~ python
from urllib.robotparser import RobotFileParser
rp = RobotFileParser()
rp.set_url('http://www.jianshu.com/robots.txt')
rp.read()#读取 robots.txt 文件并进行分析，该方法一定要调用才能有后续的内容
# 该方法传入两个参数，第一个是 User-agent，第二个是要抓取的 URL。返回的内容是该搜索引擎是否可以抓取这个 URL，返回结果是 True 或 False。
print(rp.can_fetch('*', 'http://www.jianshu.com/p/b67554025d7d'))
print(rp.can_fetch('*', "http://www.jianshu.com/search?q=python&page=1&type=collections"))
~~~

## 6.状态码与对应的条件

> 出了返回码和相应的查询条件：
>
> ```
> # 信息性状态码  
> 100: ('continue',),  
> 101: ('switching_protocols',),  
> 102: ('processing',),  
> 103: ('checkpoint',),  
> 122: ('uri_too_long', 'request_uri_too_long'),  
> 
> # 成功状态码  
> 200: ('ok', 'okay', 'all_ok', 'all_okay', 'all_good', '\\o/', '✓'),  
> 201: ('created',),  
> 202: ('accepted',),  
> 203: ('non_authoritative_info', 'non_authoritative_information'),  
> 204: ('no_content',),  
> 205: ('reset_content', 'reset'),  
> 206: ('partial_content', 'partial'),  
> 207: ('multi_status', 'multiple_status', 'multi_stati', 'multiple_stati'),  
> 208: ('already_reported',),  
> 226: ('im_used',),  
> 
> # 重定向状态码  
> 300: ('multiple_choices',),  
> 301: ('moved_permanently', 'moved', '\\o-'),  
> 302: ('found',),  
> 303: ('see_other', 'other'),  
> 304: ('not_modified',),  
> 305: ('use_proxy',),  
> 306: ('switch_proxy',),  
> 307: ('temporary_redirect', 'temporary_moved', 'temporary'),  
> 308: ('permanent_redirect',  
>    'resume_incomplete', 'resume',), # These 2 to be removed in 3.0  
> 
> # 客户端错误状态码  
> 400: ('bad_request', 'bad'),  
> 401: ('unauthorized',),  
> 402: ('payment_required', 'payment'),  
> 403: ('forbidden',),  
> 404: ('not_found', '-o-'),  
> 405: ('method_not_allowed', 'not_allowed'),  
> 406: ('not_acceptable',),  
> 407: ('proxy_authentication_required', 'proxy_auth', 'proxy_authentication'),  
> 408: ('request_timeout', 'timeout'),  
> 409: ('conflict',),  
> 410: ('gone',),  
> 411: ('length_required',),  
> 412: ('precondition_failed', 'precondition'),  
> 413: ('request_entity_too_large',),  
> 414: ('request_uri_too_large',),  
> 415: ('unsupported_media_type', 'unsupported_media', 'media_type'),  
> 416: ('requested_range_not_satisfiable', 'requested_range', 'range_not_satisfiable'),  
> 417: ('expectation_failed',),  
> 418: ('im_a_teapot', 'teapot', 'i_am_a_teapot'),  
> 421: ('misdirected_request',),  
> 422: ('unprocessable_entity', 'unprocessable'),  
> 423: ('locked',),  
> 424: ('failed_dependency', 'dependency'),  
> 425: ('unordered_collection', 'unordered'),  
> 426: ('upgrade_required', 'upgrade'),  
> 428: ('precondition_required', 'precondition'),  
> 429: ('too_many_requests', 'too_many'),  
> 431: ('header_fields_too_large', 'fields_too_large'),  
> 444: ('no_response', 'none'),  
> 449: ('retry_with', 'retry'),  
> 450: ('blocked_by_windows_parental_controls', 'parental_controls'),  
> 451: ('unavailable_for_legal_reasons', 'legal_reasons'),  
> 499: ('client_closed_request',),  
> 
> # 服务端错误状态码  
> 500: ('internal_server_error', 'server_error', '/o\\', '✗'),  
> 501: ('not_implemented',),  
> 502: ('bad_gateway',),  
> 503: ('service_unavailable', 'unavailable'),  
> 504: ('gateway_timeout',),  
> 505: ('http_version_not_supported', 'http_version'),  
> 506: ('variant_also_negotiates',),  
> 507: ('insufficient_storage',),  
> 509: ('bandwidth_limit_exceeded', 'bandwidth'),  
> 510: ('not_extended',),  
> 511: ('network_authentication_required', 'network_auth', 'network_authentication')
> ```
