---
title: scrapy_harry
date: 2016-10-02 14:51:32
tags: [spider]
---

# 创建新项目

需要创建一个项目，创建一个名为project的项目
```
scrapy startproject project
```
创建的项目结构如下
```
tree project -L 3

project/  
├── project/ #项目的Python模块，在此加入代码 
│   ├── __init__.py
│   ├── items.py
│   ├── pipelines.py
│   ├── settings.py #项目的设置文件，headers等等都在里面配置
│   └── spiders/ #放置spider代码的目录
│       └── __init__.py
└── scrapy.cfg  #配置文件
```

# 定义Item

继承`scrapy.Item`类

`Item.py`
```
import scrapy

class FirstItem(scrapy.Item):
    title = scrapy.Field()
    link = scrapy.Field()
    des = scrapy.Field()
```

# 编写第一个爬虫

继承`scrapy.Spider`类

- `name`是爬虫的名字
- `start_urls`爬虫爬取的URLS
- `parse()`爬取方法，每个初始URL完成下载后生成，`Response`对象作为唯一参数传递给该函数，该方法负责解析数据，提取数据，生成`item`和进一步要处理的URL的`Request`

`spiders/first_spider.py`
```
import scrapy

class FirstSpider(scrapy.Spider):
    name = "first"
    base_domain = "harryhei.github.io"
    start_urls = [
        "http://harryhei.github.io/tags/flask/",
        "http://harryhei.github.io/tags/django/"
    ]

    def parse(self, response):
        f = response.url.split("/")[-2]
        with open(f, "wb") as f:
            f.write(response.body)
```

制定headers和cookies参数

```
Request(url, method='GET', headers=headers, cookies=cookies)
```

# 爬取

在根目录project下使用命令
```
scrapy list #列出了所有spider
scrapy crawl first #启动了名为first的爬虫
```
爬完之后在根目录下多了两个文件：`django`和`flask`

Scrapy为每个URL创建了 `scrapy.Request` 对象，并将 `parse` 方法作为回调函数(callback)赋值给了`Request`。

`Request`对象经过调度，执行生成 `scrapy.http.Response` 对象并送回给 `parse()` 方法。

# 提取Item

selectors选择器：基于XPath和CSS
[选择器](http://scrapy-chs.readthedocs.io/zh_CN/1.0/topics/selectors.html#scrapy.selector.Selector.xpath)

```
from scrapy.selector import Selector
from scrapy.http import HtmlResponse

body = '...'
Selector(text=body).xpath('//span/text()').extract()
```

XPath举例

- `/html/head/title` ：选择HTML文档中`<head>`标签内的`<title>`元素
- `/html/head/title/text()`：选择`<title>`元素的文字
- `//td`：所有`<td>`元素
- `//div[@class="mine"]`：所有具有`class="mine"`属性的`div`元素

[xpath语法](http://www.w3school.com.cn/xpath/xpath_syntax.asp)

Selector基本方法

- `xpath()`：传入xpath表达式，返回selector list
- `css()`：传入CSS表达式，返回selector list
- `extract()`：返回unicode list
- `re()`：传入正则表达式，返回unicode list

Scrapy shell

浏览器配合分析元素，抓取内容
```
scrapy shell "http://harryhei.github.io" //获得一个response对象

response.xpath("//p[@class='article-more-link']/a/@href") //所有链接
response.xpath("//a[@class='article-title']/text()").extract() //所有标题
```

写到项目文件中

输出爬到的标题、链接和描述
```
import scrapy

class FirstSpider(scrapy.Spider):
    name = "first"
    base_domain = "harryhei.github.io"
    start_urls = [
        "http://harryhei.github.io"
    ]

    def parse(self, response):
        for sel in response.xpath("//div[@class='body-wrap']/article"):
            title = sel.xpath("div[@class='article-inner']//a/text()").extract()
            link = sel.xpath("div[@class='article-meta']//a/@href").extract()
            des = sel.xpath("div[@class='article-inner']//ul/li/a/text()").extract()
            print title[0],link[0]
            for l in des:
                print ">",l
```

使用item

`response.urljoin()`用于构造绝对路径
```
import scrapy

from project.items import FirstItem

class FirstSpider(scrapy.Spider):
    name = "first"
    base_domain = "harryhei.github.io"
    start_urls = [
        "http://harryhei.github.io"
    ]

    def parse(self, response):
        for sel in response.xpath("//div[@class='body-wrap']/article"):
            item = FirstItem()
            title = sel.xpath("div[@class='article-inner']//a/text()").extract()
            link = sel.xpath("div[@class='article-meta']//a/@href").extract()
            des = sel.xpath("div[@class='article-inner']//ul/li/a/text()").extract()
            item["title"] = title[0]
            item["link"] = response.urljoin(link[0]) #构造绝对路径
            item["des"] = " ".join(des)
            yield item
```

追踪链接

使用parse找寻链接，然后调用回调函数
```
import scrapy

from project.items import FirstItem

class FirstSpider(scrapy.Spider):
    name = "first"
    base_domain = "harryhei.github.io"
    start_urls = [
        "http://harryhei.github.io"
    ]

    def parse(self, response):
        for i in range(10):
            url = response.urljoin("/page/%s"%(i+1))
            yield scrapy.Request(url, callback=self.parse_dir_contents)

    def parse_dir_contents(self, response):
        for sel in response.xpath("//div[@class='body-wrap']/article"):
            item = FirstItem()
            title = sel.xpath("div[@class='article-inner']//a/text()").extract()
            link = sel.xpath("div[@class='article-meta']//a/@href").extract()
            des = sel.xpath("div[@class='article-inner']//ul/li/a/text()").extract()
            item["title"] = title[0]
            item["link"] = response.urljoin(link[0])
            item["des"] = " ".join(des)
            yield item
```

# 保存数据

保存为json数据
```
scrapy crawl first -o items.json
```

Python 读取json数据然后输出
```
import json

f = open("items.json")
j = json.loads(f.read())
for l in j:
    print l["title"], l["link"]
    print l["des"]
    
f.close()
```

# 参考文档
[中文文档](http://scrapy-chs.readthedocs.io/zh_CN/1.0/index.html)
