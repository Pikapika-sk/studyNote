#  Ajax

## 什么是Ajax

Ajax，全称为 Asynchronous JavaScript and XML，即异步的 JavaScript 和 XML。它不是一门编程语言，而是利用 JavaScript 在保证页面不被刷新、页面链接不改变的情况下与服务器交换数据并更新部分网页的技术。

## 基本原理

发送Ajax请求到网页更新的过程可以简单分解为 

- 发送请求
- 解析内容
- 渲染网页

微博的下拉刷新，这其实就是 JavaScript 向服务器发送了一个 Ajax 请求，然后获取新的微博数据，将其解析，并将其渲染在网页中。

## Ajax 分析方法

![image-20210130153301662](https://gitee.com/Pikapika-sk/study-note/raw/master/img/image-20210130153301662.png)

- ** x-requested-with:**XMLHttpRequest :标记了该请求是Ajax请求

- 点击XHR后在下方显示的所有请求都是Ajax请求了



# 微博Ajax实例

```python
import urllib
from urllib.parse import urlencode
from urllib.request import urlretrieve

import requests
from pyquery import PyQuery as pq

min_since_id = ''
base_url = 'https://m.weibo.cn/api/container/getIndex?'

headers = {
    'Host': 'm.weibo.cn',
    'Referer': 'https://m.weibo.cn/u/2331498495',
    'User-Agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_3) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/58.0.3029.110 Safari/537.36',
    'X-Requested-With': 'XMLHttpRequest',  # 体现Ajax的地方
}


def get_page(since_id):
    params = {
        'type': 'uid',
        'value': '2654550084',
        'containerid': '1076032654550084',
        'since_id': since_id
    }
    url = base_url + urlencode(params)

    try:
        response = requests.get(url, headers=headers)
        if response.status_code == 200:
            return response.json()
    except requests.ConnectionError as e:
        print('Error', e.args)


def get_since_id():
    global min_since_id

    now_url = 'https://m.weibo.cn/api/container/getIndex?type=uid&value=2654550084&containerid=1076032654550084'
    now_url = now_url + '&since_id=' + str(min_since_id)
    result = requests.get(now_url, headers=headers)
    json = result.json()
    items = json.get('data').get('cardlistInfo')
    # print(items)
    min_since_id = items['since_id']
    return min_since_id


def main():
    for i in range(85):
        print('第{}页'.format(i))
        result = get_page(get_since_id())
        if result:
            items = result.get('data').get('cards')
            print(items)
            for item in items:
                item = item.get('mblog')
                weibo = {}
                weibo['id'] = item.get('id')
                weibo['text'] = pq(item.get('text')).text()
                weibo['attitudes'] = item.get('attitudes_count')
                weibo['comments'] = item.get('comments_count')
                weibo['reposts'] = item.get('reposts_count')
                pics = item.get('pics')
                if pics:
                    for pic in pics:
                        large = pic['large']
                        img_url = large['url']
                        store_place = 'F:/spider_test/' + img_url[35:]
                        print(img_url)
                        urlretrieve(img_url, store_place)


if __name__ == '__main__':
    main()
```

