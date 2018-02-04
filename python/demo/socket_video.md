---
title: 远程视频
date: 2016-04-04 16:20:00
tags: [socket]
---

# 功能
实现同一局域网内单向远程视频
关键步骤是采集的BGR格式图片要转化成JPG格式后再传输
接收端将接收到的图片转化为BGR动态显示
客户端采集摄像头数据，加密并发送给服务器端
# 客户端
```
#!/usr/bin/env python
# -*- coding:utf-8 -*-

"客户端"

from socket import *
import time
import cv2
import numpy as np
import sys
import pickle,StringIO
from PIL import Image

HOST = "localhost"
PORT = 21575
ADDR = (HOST,PORT)

tcpCliSock = socket(AF_INET, SOCK_STREAM)    #创建客户套接字
try:
    tcpCliSock.connect(ADDR)    #连接服务器
except:
    print "服务器连接失败"
    sys.exit(1)

cap = cv2.VideoCapture(0)
cv2.namedWindow("v")

ret = cap.set(3,320)
ret = cap.set(4,240)

try:
    while True:
        ret, frame = cap.read()
        img = cv2.cvtColor(frame, cv2.COLOR_BGR2RGB)
        img = Image.fromarray(img)
        src = StringIO.StringIO()
        img.save(src,"JPEG")
        value = src.getvalue()
        src.close()
        value = value.replace("\n","\-n")
        tcpCliSock.sendall(value+"\n")   #发送数据
        time.sleep(0.1)
except:
    tcpCliSock.close()
```

# 服务器端
服务器端接收摄像头数据，解码并显示
```
#!/usr/bin/env python
# -*- coding:utf-8 -*-

"服务器端"

from socket import *
import time
import numpy as np
import sys,cv2
import pickle,StringIO
from PIL import Image

HOST = "localhost"
PORT = 21575
ADDR = (HOST,PORT)

tcpSerSock = socket(AF_INET, SOCK_STREAM)    #创建服务器套接字
tcpSerSock.bind(ADDR)   #地址绑定到套接字，地址是双元素元祖
tcpSerSock.listen(5)    #监听，最多允许5个客户连接到服务器

while True:
    try:
        tcpCliSock, addr = tcpSerSock.accept()  #接受客户连接

        while True:
            try:
                data = tcpCliSock.recv(1024*1024).strip()
                data = data.replace("\-n","\n")
                src = StringIO.StringIO()
                src.write(data)
                src.seek(0)
                out = Image.open(src).convert("RGB")
            except:
                continue
            cv_img = np.array(out)
            cv_img = cv2.cvtColor(cv_img, cv2.COLOR_BGR2RGB)
            cv2.imshow("v",cv_img)
            if cv2.waitKey(25) & 0xFF == ord('q'):
            	break
            src.close()
            if not data:
                print "客户端已经退出"
                break
    except KeyboardInterrupt, e:
        print "\n"
        print "byebye~"
        tcpSerSock.close()
        try:
            tcpCliSock.close()
        except:
            print "没有客户端连接"
        sys.exit(1)

```
可以在VPS上建立服务器作为中转站，实现非局域网内远程视频

参考
<http://blog.csdn.net/huhumama0/article/details/9164873>
