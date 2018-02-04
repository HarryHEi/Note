---
title: 爬图
date: 2016-03-20 19:26:19
tags: [spider]
---


# 功能
此程序通过urllib2等模块，爬取网页妹子图片

# 爬妹子图片
```
#!/usr/bin/env python
# -*- coding: utf-8 -*-  

import threading,urllib2,urllib
import os,re,string,sys,time
from bs4 import BeautifulSoup

"抓图"

countnum = 0
obj = "/home/herui/Python_Spider/"
threads = []

# 获取页面的html
def get_page(url):
        req = urllib2.Request(url)
        try:
                html = urllib2.urlopen(req)
                return html
        except urllib2.URLError, e:
            if hasattr(e, "code"):
                print "HTTP error: " + str(e.code)
            elif hasattr(e, "reason"):
                print e.reason

#获取分类链接
def get_fenlei_link(url, html):
    soup = BeautifulSoup(html, 'lxml')
    links = soup.find_all(href = re.compile("/mm/"))[0:5]
    fenlei_links = []
    for link in links:
        longlink = str(link).split('\"')[1]
        fenlei_links.append(url+longlink)
    return fenlei_links

#获得页面所有图片的链接
def get_pic_links(url):
    req = urllib2.Request(url)
    try:
        html = urllib2.urlopen(req)
    except urllib2.URLError, e:
        if hasattr(e, "code"):
            print "HTTP error: " + str(e.code)
        elif hasattr(e, "reason"):
            print e.reason
    soup = BeautifulSoup(html, 'lxml')
    links = soup.find_all(src = re.compile(".jpg"))
    pic_url = []
    for link in links:
        pic_url.append(re.findall("http:.*?.jpg",str(link)))
    return pic_url


#获得每个图包的链接
# "/mm/xxxxx/xxxxxxxxxxxxxx.html"
def get_pic_pack_link(page_index):
    html = urllib2.urlopen(page_index).read()
    links = re.findall("/mm/.*?html", str(html))[1:]
    return links

#抓取一个图包里面的十个图片
def get_pack_pic(page_index,index_num, path, packlink ,pack_num):
    global countnum
    global obj
    print "开始下载第%d个图包"%pack_num
    for i in range(1,11): #每个图抱抓10个
        name = obj+str(path)+'/'+str(index_num)+str(pack_num)+str(string.zfill(i,5))+".jpg"
        if i <= 1:
            packurl = str(re.findall("http://.*/", str(page_index))[0]) + str(str(str(packlink).split(".")[0]).split("/")[-1]) + ".html"
        else:
            packurl = str(re.findall("http://.*/", str(page_index))[0]) + str(str(str(packlink).split(".")[0]).split("/")[-1]) + "-" +str(i) + ".html"
        try:
            packhtml = urllib2.urlopen(packurl).read()
            packpicurl = str(str(re.findall("http://22mm-img.*?_640.jpg", packhtml)).split("=")[-1]).split(",")[-1]
            packpicurl = re.findall("http://.*?.jpg", packpicurl)[0]
            packpicurl = packpicurl.replace("big","pic",1)
        except:
            continue
        try:
            urllib.urlretrieve(packpicurl,name)
        except:
            print "图片下载失败"
            continue
        countnum += 1
        print "%s"%str(name) + "下载成功"

#开始
def start():
        print 'Waiting............'
        url='http://www.22mm.cc'
        html=get_page(url)
        fenlei_links = get_fenlei_link(url, html)
        for link in fenlei_links:
            pagenum = 0
            while pagenum <= 5:
                pagenum += 1
                if pagenum == 1:
                    path=unicode(str(link).split("/")[-2],'utf8')
                    if not os.path.exists(path):
                        os.mkdir(path)
                    page_index = link+"index"+".html"
                else:
                    page_index = link+"index_"+str(pagenum)+".html"
                packlinks = get_pic_pack_link(page_index) #每个分类每页每个图包首地址
                pack_num = 0
                for packlink in packlinks:
                    thread = threading.Thread(target=get_pack_pic,args=(page_index, pagenum, path, packlink, pack_num))
                    thread.setDaemon(True)
                    thread.start()
                    threads.append(thread)
                    pack_num += 1
                    print "开始第%s个线程"%pack_num

if __name__ == "__main__":
    try:
        start()
        for thread in threads:
            time.sleep(1)
            while thread.join():
                pass
        print "一共下载了%d个图片"%countnum
    except KeyboardInterrupt, e :
        print "一共下载了%d个图片"%countnum
        for thread in threads:
            sys.exit(1)
```