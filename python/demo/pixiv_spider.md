---
title: pixiv抓图
date: 2016-04-28 12:48:53
tags: [python,spider]
---

# 简述
因为在学JavaScript，所以准备随便抓些图片放在网页上，然后锁定了pixiv，用urllib2抓链接发现不能下载，一直403
然后查了下，发现wget的神奇功能，直接用os.system调用

# 抓图代码
```
#!/usr/bin/env python
# -*- coding: utf-8 -*-  

import threading,urllib2,urllib
import os,re,string,sys,time
from bs4 import BeautifulSoup
import cookielib

"使用wget抓取pixiv的图片"

url = "http://spotlight.pics/en/a/1196"

header = {    
    "User-Agent":"Mozilla/5.0 (X11; Ubuntu; Linux i686; rv:45.0) Gecko/20100101 Firefox/45.0",
    "Accept":"text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8"
 }

#获得页面所有图片的链接
def get_pic_links(url):
    cookie = cookielib.CookieJar()
    cookie_handler = urllib2.HTTPCookieProcessor(cookie)
    opener = urllib2.build_opener(cookie_handler)
    req = urllib2.Request(url, headers=header)
    try:
        html = opener.open(req)
    except urllib2.URLError, e:
        if hasattr(e, "code"):
            print "HTTP error: " + str(e.code)
        elif hasattr(e, "reason"):
            print e.reason
        else:
            print "unknow err"
    soup = BeautifulSoup(html, 'lxml')
    links = soup.find_all(src = re.compile(".jpg"))
    pic_url = []
    for link in links:
        pic_url.append(re.findall("http:.*?.jpg",str(link)))
    return pic_url

def get_pic(u):
    command = "wget --referer=" + "\"" + str(u[0]) + "\""  + " -p " + "\"" + str(u[0]) + "\""
    os.system(command)

if __name__ == "__main__":
    urls = get_pic_links(url)
    for u in urls:
        get_pic(u)
```

# 抓下来的图片统一改名
```
#!/usr/bin/env python
#-*-coding:utf-8-*-

"批量修改文件名"

import os
import sys

url = "/home/herui/Python_Spider/"

i=0

for root,dirs,files in os.walk(url):
    for f in files:
        if f.split(".")[-1] == "jpg":
            filename = os.path.join(root,f)
            newname = os.path.join("/home/herui/Python_Spider/pic","pic"+str(i)+".jpg")
            i=i+1
            os.rename(filename, newname)

print "ok"
```

# 如果只要大图
点开爬到的内容可以发现，大图都在 480x960 文件夹里面，再添加一句
```
#!/usr/bin/env python
#-*-coding:utf-8-*-

"批量修改文件名"

import os
import sys

url = "/home/herui/Python_Spider/"

i=0

for root,dirs,files in os.walk(url):
    for f in files:
        if f.split(".")[-1] == "jpg" and ("480x960" in root):
            filename = os.path.join(root,f)
            newname = os.path.join("/home/herui/Python_Spider/pic2","pic"+str(i)+".jpg")
            i=i+1
            os.rename(filename, newname)

print "ok"
```

# 参考
https://prinzeugen.net/use-multiple-threads-to-download-pixiv-anime/
