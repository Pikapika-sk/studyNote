# BeautiflSoup

## 1.Beautiful Soup的各类解析器

~~~ python
from bs4 import BeautifulSoup
soup = BeautifulSoup('<p>data</P>', 'html.parser')
~~~

![image-20210123104052105](https://gitee.com/Pikapika-sk/study-note/raw/master/img/image-20210123104052105.png)

## 2.BeautifulSoup类的基本元素

![image-20210123104450231](https://gitee.com/Pikapika-sk/study-note/raw/master/img/image-20210123104450231.png)

![image-20210123110044380](https://gitee.com/Pikapika-sk/study-note/raw/master/img/image-20210123110044380.png)

## 3.基于bs4库的HTML内容遍历方法

![image-20210123111527391](https://gitee.com/Pikapika-sk/study-note/raw/master/img/image-20210123111527391.png)

### 下行遍历

![image-20210123113105478](https://gitee.com/Pikapika-sk/study-note/raw/master/img/image-20210123113105478.png)

~~~ python
for child in soup.body.children:
	print(child)  #遍历儿子节点
    
for child in soup.body.descendants:
    print(child)		#遍历子孙节点
~~~

### 上行遍历

![image-20210123113247224](https://gitee.com/Pikapika-sk/study-note/raw/master/img/image-20210123113247224.png)

~~~ python
for parent in soup.a.parents:
    if parent is None:
        print(parent)
    else:
        print(parent.name)
~~~

### 平行遍历

![image-20210123113411591](https://gitee.com/Pikapika-sk/study-note/raw/master/img/image-20210123113411591.png)

![image-20210123113425219](https://gitee.com/Pikapika-sk/study-note/raw/master/img/image-20210123113425219.png)

~~~ python
for sibling in soup.a.next_sibling:
    print(siibing)  #遍历后续节点
 
for sibling in soup.a.previous_sibling:
    print(sibling)   #遍历前序节点
~~~

## 4.基于bs4库的HTML格式输出

### bs4库的prettify()方法

- prettify() 为HTML文本<>及其内容增加更加'\n'
- .prettify()可用于标签，方法：.prettify()
- bs4库将任何HTML输入都变成utf‐8编码

~~~ python
soup.prettify()
~~~

## 5.基于BS4库的HTML内容查找方法

![image-20210123133258265](https://gitee.com/Pikapika-sk/study-note/raw/master/img/image-20210123133258265.png)

![image-20210123133301829](https://gitee.com/Pikapika-sk/study-note/raw/master/img/image-20210123133301829.png)

![image-20210123133309305](https://gitee.com/Pikapika-sk/study-note/raw/master/img/image-20210123133309305.png)

## 6.总结

- 推荐使用 LXML 解析库，必要时使用 html.parser。

- 节点选择筛选功能弱但是速度快。

- 建议使用 find、find_all 方法查询匹配单个结果或者多个结果。

- 如果对 CSS 选择器熟悉的话可以使用 select 选择法。

	-----

# 中国大学排名定向爬虫

~~~ python
import requests
from bs4 import BeautifulSoup
import bs4

def getHTMLText(url):
    try:
        r = requests.get(url, timeout=30)
        r.raise_for_status()
        r.encoding = r.apparent_encoding
        return r.text
    except:
        return ""

def fillUnivList(ulist, html):
    soup = BeautifulSoup(html, "html.parser")
    for tr in soup.find('tbody').children:
        if isinstance(tr, bs4.element.Tag):
            tds = tr('td')
            ulist.append([tds[0].string, tds[1].string, tds[3].string])

def printUnivList(ulist, num):
    tplt = "{0:^10}\t{1:{3}^10}\t{2:^10}"
    print(tplt.format("排名","学校名称","总分",chr(12288)))
    for i in range(num):
        u=ulist[i]
        print(tplt.format(u[0],u[1],u[2],chr(12288)))
    
def main():
    uinfo = []
    url = 'https://www.zuihaodaxue.cn/zuihaodaxuepaiming2016.html'
    html = getHTMLText(url)
    fillUnivList(uinfo, html)
    printUnivList(uinfo, 20) # 20 univs
main()
~~~

