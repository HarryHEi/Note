---
title: wxBot微信机器人框架
date: 2016-09-29 16:29:47
tags: [python]
---

# 前言

> wxBot 是用Python包装Web微信协议实现的微信机器人框架

[wxBot](https://github.com/liuwons/wxBot)项目地址

新建一个虚拟环境
```
$ mkdir mywxbot
$ cd mywxbot/
$ virtualenv venv
$ source venv/bin/activate
```

# 安装依赖

```
pip install requests
pip install pyqrcode
pip install pypng
pip install Pillow
```

# 代码

自动回复

```
#!/usr/bin/env python
# coding: utf-8

import time
from wxbot import *

class MyWXBot(WXBot):
    def handle_msg_all(self, msg):
        if msg['msg_type_id'] == 4 and msg['content']['type'] == 0:
            self.send_msg_by_uid(u'hi', msg['user']['id']) //这里自动回复消息hi
            //还有很多方式，在github中有写

def main():
    bot = MyWXBot()
    bot.DEBUG = True
    bot.run()

if __name__ == '__main__':
    main()
```

# 爬虫加微信接口

- 功能是微信收到消息之后，request爬百度对应的图片，然后回过去

- 项目地址[mywxbot](https://github.com/HarryHEi/mywxbot)

- 发送图片的时候如果设置为中文名称就会出错

- 首先用微信接受字符串，注意要把unicode编码转成str

- 百度的图片资源可能会403

- 图片名是用的随机字符串，也可以用名称的md5值，不过会出现图片重复，这样可以避免重复下载

- 下载图片用的是request文档的推荐，也可以用wget，不过之前试的时候经常403

- 爬到的图片不是高清的

- 总之，也算基本完成预期吧

```
#!/usr/bin/env python
# coding: utf-8

import time
from wxbot import *
import requests
import os,re,string,sys,time
import cookielib
import random
import md5

"百度爬图的微信接口"

url = "http://image.baidu.com/search/index?tn=baiduimage&ps=1&ct=201326592&lm=-1&cl=2&nc=1&ie=utf-8&word="

#获得页面所有图片的链接
def get_pic_links(url):
    req = requests.get(url)
    html = req.text
    links = re.findall(r'"thumbURL":"(.*?.jpg)"', html) #小图
    sourcelinks = re.findall(r'"objURL":"(.*?)"', html) #原图
    return links, sourcelinks

#使用wget下载
def wget_pic(u, name):
    referer = re.findall(r'//(.*?)/',u)[0]
    command = 'wget --referer="%s" -O "%s"  "%s"'%(referer, name, u)
    print "Command is: %s"%(command)
    os.system(command)

class MyWXBot(WXBot):
    def handle_msg_all(self, msg):
        if msg['msg_type_id'] == 4 and msg['content']['type'] == 0:
            user_input = msg["content"]["data"]
            user_input = user_input.encode("utf-8") #Unicode => str
            #m = md5.new()
            #m.update(user_input)
            #md5name =  m.hexdigest()
            md5name = random_str(10)
            name = ''.join([md5name, '.jpg'])
            #name = name.encode("utf-8") #Unicode => str
            urls, sourceurls = get_pic_links(url+ user_input)
            if len(urls)<1:
                self.send_msg_by_uid(u'并没有搜到啥', msg['user']['id'])
                return 0
            else:
                self.send_msg_by_uid(u'请稍等', msg['user']['id'])
                i = 0
                while i < 20:
                    i = i + 1;
                    if not os.path.exists(name):
                        num = random.randint(1,len(urls)-1)
                        #下载图片，命名为name
                        try:
                            headers = {
                                "User-Agent":"Mozilla/5.0 (X11; Ubuntu; Linux i686; rv:45.0) Gecko/20100101 Firefox/45.0",
                                "Accept":"text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8"
                            }
                            referer = re.findall(r'//(.*?)/',urls[num])[0]
                            headers["referer"] = referer
                            r = requests.get(urls[num], headers = headers)
                            if str(r.status_code)[0] == "4":
                                print("未下载成功：", urls[num])
                                continue
                            with open(name, 'wb') as fd:
                                for chunk in r.iter_content():
                                    fd.write(chunk)
                        except:
                            print "图片下载错误"
                            continue
                    #发送图片
                    try:
                        self.send_img_msg_by_uid(name, msg['user']['id'])
                        self.send_msg_by_uid(u'原图链接', msg['user']['id'])
                        self.send_msg_by_uid(sourceurls[num], msg['user']['id'])
                        break
                    except:
                        self.send_msg_by_uid(u'并没有搜到啥', msg['user']['id'])
                        continue

def random_str(randomlength=8):
    str = ''
    chars = 'AaBbCcDdEeFfGgHhIiJjKkLlMmNnOoPpQqRrSsTtUuVvWwXxYyZz0123456789'
    length = len(chars) - 1
    r = random.Random()
    for i in range(randomlength):
        str+=chars[r.randint(0, length)]
    return str

def main():
    bot = MyWXBot()
    bot.DEBUG = True
    bot.SCHEDULE_INTV = 20
    bot.conf['qr'] = 'tty'
    bot.run()

if __name__ == '__main__':
    main()

```

# 参考
[微信树莓派](http://blog.just4fun.site/pi-wechat.html)

使用`flask`提供`api`，当接收到微信消息时，使用`requests.get(url)`访问`api`，好像挺有意思

可以做一个类似弹幕的东西，用微信发送文本信息，然后实时更新到网页上