---
title: 聊天室
date: 2016-03-23 14:59:26
tags: [socket]
---


# 简介
聊天室就是允许多人一起聊天，类似于QQ群的结构
主要原理就是用户发送消息给服务器，再由服务器发送消息给所有其他人
主要包括两个部分:服务器和客户端
# 服务器部分
服务器需要同时监听多个客户端的消息，可以使用select模块
注意在windows不能直接使用select
select的三个参数分别代表read,write,error
定义一个数组connection_lis用来存放连接的客户端套接字
```
connection_lis = [tcpSerSock]
sread, swrite, serror = select.select(connection_lis,[],[])
```
当select有返回值时，如果是服务器的socket说明有客户端连接
如果是其他客户端的socket，说明客户端发送数据给服务器
代码结构像这样
```
for s in sread:
    #新用户连接
    if s == tcpSerSock:
        pass
    #用户数据
    else:
       pass
```
想要实现聊天，还必须要将用户的消息发送给其他用户
因为之前定义了一个用户数组connection_lis
直接使用for循环轮询发送消息
```
#广播消息
def broadcast_data(sock, message):
    #逐一发送
    for s in connection_lis:
        if (s != tcpSerSock) and (s != sock): #除了服务器自身和发送者
            try:
                if isinstance(message, bytes):
                    s.send(message+"\r\n")
                else:
                    s.send(message.encode()+"\r\n")
            except:
                try:
                    print "不能给客户端[ %s ]发送消息" %s.getpeername()
                except:
                    print "不能识别客户端"
                s.close()   #如果发送失败，断开连接并删除
                connection_lis.remove(s)
```
用户退出
注意在broadcast_data之后再删除数组里面的用户socket
```
def client_close(sock):
    broadcast_data(sock, "[ %s ]退出了"%str(sock.getpeername()))
    print "[ %s ]退出了"%str(sock.getpeername())
    connection_lis.remove(sock)
    sock.close()
```
全部代码
```
#!/usr/bin/env python
# -*- coding:utf-8 -*-

"聊天服务器"

from socket import *
import sys
import select

HOST = ""
PORT = 21567
BUFSIZ = 1024
ADDR = (HOST,PORT)

tcpSerSock = socket(AF_INET, SOCK_STREAM)    #创建服务器套接字
tcpSerSock.setsockopt(SOL_SOCKET, SO_REUSEADDR, 1)  #地址复用
tcpSerSock.bind(ADDR)   #地址绑定到套接字，地址是双元素元祖
tcpSerSock.listen(5)    #监听，最多允许5个客户连接到服务器

connection_lis = [tcpSerSock]

#广播消息
def broadcast_data(sock, message):
    #逐一发送
    for s in connection_lis:
        if (s != tcpSerSock) and (s != sock): #除了服务器自身和发送者
            try:
                if isinstance(message, bytes):
                    s.send(message+"\r\n")
                else:
                    s.send(message.encode()+"\r\n")
            except:
                try:
                    print "不能给客户端[ %s ]发送消息" %s.getpeername()
                except:
                    print "不能识别客户端"
                s.close()   #如果发送失败断开连接，删除
                connection_lis.remove(s)

def client_close(sock):
    broadcast_data(sock, "[ %s ]退出了"%str(sock.getpeername()))
    print "[ %s ]退出了"%str(sock.getpeername())
    connection_lis.remove(sock)
    sock.close()

print "聊天室: ", ADDR

print "wait..."

while True:
    #监听
    try:
        sread, swrite, serror = select.select(connection_lis,[],[])
    except KeyboardInterrupt, e:
        print " "
        print "byebye~"
        sys.exit(1)

    for s in sread:
        #新用户连接
        if s == tcpSerSock:
            try:
                clientsock, addr = s.accept()
                print "...新用户连接: ", addr
                connection_lis.append(clientsock)
                print "添加到用户"
                broadcast_data(tcpSerSock, "[%s]加入聊天室\r\n" %str(addr))
                print "当前人数:", str(len(connection_lis) - 1)
            except:
                print "连接失败"
        #用户数据
        else:
            try:
                data = s.recv(BUFSIZ)
                if data.strip() == "":
                    client_close(s)
                    continue
                else:
                    broadcast_data(s, "[ %s ]: %s" %(s.getpeername(), str(data)))
            except:
                print "用户数据接收失败"
```
# 客户端部分
客户端主要需要做两件事:监听服务器发送的数据和发送数据给服务器
定义一个数组rlist，存放标准输入和套接字
```
rlist = [sys.stdin, clientserver]
```
使用select监听
```
sread, swrite, serror = select.select(rlist, [], [])
```
select返回数据，如果是clientserver，说明是服务器会送了消息
如果时标准输入，说明时用户输入了数据
全部代码如下
```
#!/usr/bin/env python
# -*- coding:utf-8 -*-

"聊天客户端"

from socket import *
import select
import sys
import time

HOST = "localhost"
PORT = 21567
BUFSIZ = 1024
ADDR = (HOST,PORT)

def prompt():
    sys.stdout.write('<You>')
    sys.stdout.flush()

try:
    clientserver = socket(AF_INET, SOCK_STREAM)
    clientserver.connect(ADDR)
except:
    print "服务器连接失败"
    sys.exit(1)

print "成功连接服务器"

while True:
    rlist = [sys.stdin, clientserver]
    try:
        sread, swrite, serror = select.select(rlist, [], [])
    except KeyboardInterrupt, e:
        print " "
        print "byebye~"
        sys.exit(1)
    for s in sread:
        #接收到消息
        if s == clientserver:
            data = s.recv(BUFSIZ)
            if not data:
                print "服务器没有反应"
                sys.exit(1)
            #显示消息
            else:
                sys.stdout.write(data)
                prompt()

        #发送消息
        else:
            try:
                message = sys.stdin.readline()
                if not message.strip():
                    clientserver.send(message)
                    print "正在退出"
                    time.sleep(1)
                    sys.exit(1)
                clientserver.send(message)
                prompt()
            except KeyboardInterrupt, e:
                print "byebye~"
                sys.exit(1)

```
# 参考
<http://www.cnblogs.com/hazir/p/python_chat_room.html>