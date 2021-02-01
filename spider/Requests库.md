# Requests库

## 常用的7种方法

![image-20210122174029872](https://gitee.com/Pikapika-sk/study-note/raw/master/img/image-20210122174029872.png)

## 函数原型

![image-20210122174101392](https://gitee.com/Pikapika-sk/study-note/raw/master/img/image-20210122174101392.png)

![image-20210122174428033](https://gitee.com/Pikapika-sk/study-note/raw/master/img/image-20210122174428033.png)

![image-20210122174439730](https://gitee.com/Pikapika-sk/study-note/raw/master/img/image-20210122174439730.png)

## Response对象的属性

![image-20210122174132797](https://gitee.com/Pikapika-sk/study-note/raw/master/img/image-20210122174132797.png)

### Response的编码

![image-20210122174204718](https://gitee.com/Pikapika-sk/study-note/raw/master/img/image-20210122174204718.png)

## 爬取网页通用代码框架

~~~ python
import requests


def getHTMLText(url):
    try:
        r = requests.get(url, timeout=30)
        r.raise_for_status()
        r.encoding = r.apparent_encoding

        return r.text
    except:
        return "产生异常"


if __name__ == "__main__":
    url = "http://www.baidu.com"
    print(getHTMLText(url))

~~~

## 进阶用法

### 1.文件上传

```python
import requests

files = {'file': open('favicon.ico', 'rb')}
r = requests.post('http://httpbin.org/post', files=files)
print(r.text)
```

### 2.Cookies

#### 获取 Cookies 的过程

```python
import requests

r = requests.get('https://www.baidu.com')
print(r.cookies)
for key, value in r.cookies.items():
    print(key + '=' + value)
```

#### headers中添加cookies

~~~ python
import requests
headers = {
    'Cookie': 'q_c1=31653b264a074fc9a57816d1ea93ed8b|1474273938000|1474273938000; d_c0="AGDAs254kAqPTr6NW1U3XTLFzKhMPQ6H_nc=|1474273938"; __utmv=51854390.100-1|2=registration_date=20130902=1^3=entry_date=20130902=1;a_t="2.0AACAfbwdAAAXAAAAso0QWAAAgH28HQAAAGDAs254kAoXAAAAYQJVTQ4FCVgA360us8BAklzLYNEHUd6kmHtRQX5a6hiZxKCynnycerLQ3gIkoJLOCQ==";z_c0=Mi4wQUFDQWZid2RBQUFBWU1DemJuaVFDaGNBQUFCaEFsVk5EZ1VKV0FEZnJTNnp3RUNTWE10ZzBRZFIzcVNZZTFGQmZn|1474887858|64b4d4234a21de774c42c837fe0b672fdb5763b0',
    'Host': 'www.zhihu.com',
    'User-Agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/53.0.2785.116 Safari/537.36',
}
r = requests.get('https://www.zhihu.com', headers=headers)
print(r.text)
~~~

### 3. 会话维持

Session 在平常用得非常广泛，可以用于模拟在一个浏览器中打开同一站点的不同页面

~~~ python
import requests

s = requests.Session()
s.get('http://httpbin.org/cookies/set/number/123456789')
r = s.get('http://httpbin.org/cookies')
print(r.text)
~~~

### 4.SSL验证

把 verify 参数设置为 False

~~~ python
import requests
from requests.packages import urllib3

urllib3.disable_warnings()  # 忽略警告
response = requests.get('https://www.12306.cn', verify=False) # 取消证书认证
print(response.status_code)
~~~

指定本地证书用于客户端证书，这可以是单个文件（包含密钥和证书）或一个包含两个文件路径的元组：

~~~ python 
import requests

response = requests.get('https://www.12306.cn', cert=('/path/server.crt', '/path/key'))
print(response.status_code)
~~~

### 5. 代理设置

```python
import requests

proxies = {
  'http': 'http://10.10.1.10:3128',
  'https': 'http://10.10.1.10:1080',
}
requests.get('https://www.taobao.com', proxies=proxies)

```

### 6. 超时设置

```python
import requests

r = requests.get('https://www.taobao.com', timeout=1)
print(r.status_code)

r  = requests.get('https://www.taobao.com', timeout=(5, 30)) # 5:链接事件，  30：读取时间                                
```

### 7. 身份认证

```python
import requests

r = requests.get('http://localhost:5000', auth=('username', 'password'))
print(r.status_code)
```

#### OAuth1 认证的方法如下：

```python
import requests
from requests_oauthlib import OAuth1

url = 'https://api.twitter.com/1.1/account/verify_credentials.json'
auth = OAuth1('YOUR_APP_KEY', 'YOUR_APP_SECRET',
              'USER_OAUTH_TOKEN', 'USER_OAUTH_TOKEN_SECRET')
requests.get(url, auth=auth)
```



# 猫眼电影爬取

~~~ python
import json
import requests
from requests.exceptions import RequestException
import re
import time

def get_one_page(url):
    try:
        headers = {'User-Agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_3) AppleWebKit/537.36 (KHTML, like '
                                 +'Gecko) Chrome/65.0.3325.162 Safari/537.36'}
        response = requests.get(url, headers=headers)
        if response.status_code == 200:
            return response.text
        return None
    except RequestException:
        return None


def parse_one_page(html):
    pattern = re.compile('<dd>.*?board-index.*?>(\d+)</i>.*?data-src="(.*?)".*?name"><a'
+ '.*?>(.*?)</a>.*?star">(.*?)</p>.*?releasetime">(.*?)</p>'
+ '.*?integer">(.*?)</i>.*?fraction">(.*?)</i>.*?</dd>', re.S)
    items = re.findall(pattern, html)
    for item in items:
        yield {'index': item[0],
            'image': item[1],
            'title': item[2],
            'actor': item[3].strip()[3:],
            'time': item[4].strip()[5:],
            'score': item[5] + item[6]
        }

def write_to_file(content):
    with open('result.txt', 'a', encoding='utf-8') as f:
        f.write(json.dumps(content, ensure_ascii=False) + '\n')

def main(offset):
    url = 'http://maoyan.com/board/4?offset=' + str(offset)
    html = get_one_page(url)
    for item in parse_one_page(html):
        print(item)
        write_to_file(item)

if __name__ == '__main__':
    for i in range(10):
        main(offset=i * 10)
        time.sleep(3)

main()
~~~



