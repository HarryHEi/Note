---
title: requests 爬虫
date: 2016-05-09 10:16:39
tags: [spider]
---

# 基础
## 传递参数
```
import requests

param = {'page':'2'} 
r =requests.get("http://119.29.137.50",params=param)
```
## 响应内容
```
import requests
from PIL import Image
from StringIO import StringIO

r =requests.get("http://119.29.137.50")
print r.text #打印内容

r =requests.get("http://7n.byherui.com/avatar.jpg")
i = Image.open(StringIO(r.content)).save("temp.jpg","JPEG") #数据流形式打开并保存
```
## json
使用内置解码器
```
import requests

param = {"ak":"asd","location":"北京","output":"json"} 
r =requests.get("http://api.map.baidu.com/telematics/v3/weather",params=param)
print r.json()
```
访问原始套接字
```
import requests

param = {"ak":"asd","location":"北京","output":"json"} 
r =requests.get("http://api.map.baidu.com/telematics/v3/weather",params=param,stream=True)
print r.raw.read()
```
文本流保存文件
```
import requests

param = {"ak":"asd","location":"北京","output":"json"} 
r =requests.get("http://api.map.baidu.com/telematics/v3/weather",params=param,stream=True)

with open("temp.txt", 'wb') as fd:
    for chunk in r.iter_content():
        fd.write(chunk)
```
## 请求头和post数据
添加一个字典
```
import requests

data = {
    "name":"Python",
    "submit":"Submit"
}

header = {
    "User-Agent":"Mozilla/5.0 (X11; Ubuntu; Linux i686; rv:46.0) Gecko/20100101 Firefox/46.0"
}

r =requests.post("http://119.29.137.50",headers=header,data=data)
print r.text
```
## cookies
```
import requests

r =requests.get("http://119.29.137.50")
print r.cookies
```
发送cookies到服务器
```
import requests

cookies = dict(cookies_are='working')
r =requests.get("http://119.29.137.50",cookies=cookies)
print r.text
```
## 超时
```
import requests

r =requests.get("http://119.29.137.50",timeout=0.001)
print r.text
```
# 高级用法
## 会话对象
会话对象能够保持某些参数，比如cookies

## beautifusoup
用beautifulsoup处理抓到的页面
按类查找标签
```
import requests
from bs4 import BeautifulSoup

html = s.get(url)
soup = BeautifulSoup(html.text, "lxml")
links = soup.find_all("span",class_="rating_num")
```
