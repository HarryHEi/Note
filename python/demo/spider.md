---
title: python_spider
date: 2016-03-15 19:45:06
tags: [python,spider]
---

# 爬虫入门
```
#!/usr/bin/env python
# -*- coding:utf-8 -*-
"这是个小爬虫"

import urllib2

response = urllib2.urlopen("http://herui.blog-harryx.cn")

print response.read()
```
直接调用urlopen函数返回一个respons对象，并用.read()函数操作这个respon对象
```
request = urllib2.Request("http://herui.blog-harryx.cn")
response = urllib2.urlopen(request)
```
使用Request()函数构造request，可以额外发送其他数据或信息
这里我们先设置为空字典
```
from urllib2 import Request, urlopen, URLError, HTTPError  
import sys

header = {    

 }

request = Request("http://herui.blog-harryx.cn", headers=header)

try:
    response = urlopen(request)

except URLError, e:
    if hasattr(e, "code"):
        print "HTTP error: " + str(e.code)
    elif hasattr(e, "reason"):
        print e.reason

    sys.exit(1)

print response.read()
```
这里使用try-except语句检验错误
要注意URLError包含HTTPError，要先检验HTTPError
或者使用hasattr()函数检验类是否包含指定参数
这里向服务器发出的请求是GET请求

```
from urllib2 import Request, urlopen, URLError, HTTPError  
from urllib import urlencode
import sys

url = "http://localhost:5000"

values = {
    "name":"harryx"
}

header = {    

 }

data = urlencode(values)
request = Request(url,data=data, headers=header)

try:
    response = urlopen(request)

except URLError, e:
    if hasattr(e, "code"):
        print "HTTP error: " + str(e.code)
    elif hasattr(e, "reason"):
        print e.reason

    sys.exit(1)

print response.read()
```
加入data参数，向本地服务器发出请求
```
127.0.0.1 - - [16/Mar/2016 15:27:06] "POST / HTTP/1.1" 200 -
```
发出的是POST请求
```
 ./spider.py > hello.html
```
打开网页，可以看到对话框填入了字符串“harryx”
## geturl()
```
response.geturl()
```
geturl()用来获取真实的URL，urlopen可能会HTTP重定向
## info()
```
print response.info()
```
返回服务器的响应报文首部，包含获取网页的信息
```
Server: GitHub.com
Content-Type: text/html; charset=utf-8
Last-Modified: Sun, 13 Mar 2016 08:07:00 GMT
Access-Control-Allow-Origin: *
Expires: Tue, 15 Mar 2016 13:13:57 GMT
Cache-Control: max-age=600
X-GitHub-Request-Id: 17EB2B31:0345:1A183C4:56E8083D
Content-Length: 27069
Accept-Ranges: bytes
Date: Wed, 16 Mar 2016 11:18:33 GMT
Via: 1.1 varnish
Age: 539
Connection: close
X-Served-By: cache-ams4127-AMS
X-Cache: HIT
X-Cache-Hits: 1
X-Timer: S1458127113.611546,VS0,VE0
Vary: Accept-Encoding
X-Fastly-Request-ID: ef7713a01a3cf84afdd68ff2640410d72a1774c1

```
## 代理服务器
使用代理服务器的方法如下
很好很强大
```
proxy_handler = ProxyHandler({"http" :'http://121.193.143.249:80'})  
null_proxy_handler = ProxyHandler({})  
enable_prosy = False

if enable_prosy:
    opener = build_opener(proxy_handler)
else:
    opener = build_opener(null_proxy_handler)

data = urlencode(values)
request = Request(url,data=None, headers=header)
response = opener.open(request,timeout=20)
```
## cookielib
```
import cookielib
cookie = cookielib.CookieJar()
cookie_handler = HTTPCookieProcessor(cookie)
opener = build_opener(cookie_handler)
```
注意的是,如果同时使用到代理服务器,需要同时添加proxy_handle
```
opener = build_opener(cookie_handler, proxy_handler)
```
# 爬自己本地的flask写的web
## 用httpfox查看自己发送的包

> csrf_token:1458359238##98ca41f600896093e2e7b12e488d8cd9eb74c3db
> name:lalala
> submit:Submit

所以我定义一个字典
```
values = {
    "csrf_token":None,
    "name":"Python",
    "submit":"Submit"
}
```
csrf_token是个随机值，第一次访问页面时获得，所以后面需要用正则表达式获取，
先不管。
查看headers里面的内容，照写
```
header = {    
    "User-Agent":user_agent,
    "Host":"localhost:5000",
    "Accept":"text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8",
    "Accept-Language":"en-US,en;q=0.5",
    "Accept-Encoding":"gzip, deflate"
 }
```
header里面还有cookie这一项，所以用cookielib库
```
import cookielib

cookie = cookielib.CookieJar()
opener = urllib2.build_opener(urllib2.HTTPCookieProcessor(cookie))
request = urllib2.Request(url, headers=header)
response = opener.open(request,timeout=20)
```
这里先读取一次页面，获得cookie，同时用正则表达式获得csrf_token的值
```
values["csrf_token"] = str(re.findall('\d{10}##\w*',response.read())).split("\'")[1]
```
然后使用urllib库里面的urlencode对values的值进行编码
```
data = urllib.urlencode(values)
```
最后读取一次页面，同时上传data数据
```
try:
    request = urllib2.Request(url,data=data, headers=header)
    response = opener.open(request,timeout=20)

except urllib2.URLError, e:
    if hasattr(e, "code"):
        print "HTTP error: " + str(e.code)
    elif hasattr(e, "reason"):
        print e.reason

    sys.exit(1)

print response.read()
```
可以在web端看到一次POST
```
127.0.0.1 - - [19/Mar/2016 10:47:26] "POST / HTTP/1.1" 302 -
```
```
<div class="page-header">
    <h1>Hello, Python!</h1>
    
    <p>Pleased to meet you!</p>
    
</div>

```
成功

# 参考

 <http://blog.csdn.net/column/details/why-bug.html>