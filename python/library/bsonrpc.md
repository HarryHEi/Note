---
title: rpc远程调用
date: 2017-03-23 15:39:37
tags: [python]
---

# 简介

RPC（Remote Procedure Call Protocol）是基于TCP或者RPC等传输协议的远程过程调用协议。

# 使用

参考：
	[github示例](https://github.com/seprich/py-bson-rpc)
	[文档](http://seprich.github.io/py-bson-rpc/index.html)

python的`bsonrpc`库通过类装饰器的方式构造RPC服务

- `@service_class`装饰RPC服务类
- `@request`装饰提供RPC调用的函数
- `@rpc_request`增加一个JSONRpc/BSONRpc对象作为参数
- `@notification`用于通知，有个字符串参数

两种方式通知：

```
@rpc_request
def long_process(self, rpc, a, b, c):
	print(rpc.client_info)
	# 2 ways to send notifications:
	rpc.invoke_notification('report_progress', 'Stage 1')
	client = rpc.get_peer_proxy()
	client.n.report_progress('Stage 2')
```

构建一个RPC类，然后将创建好的套接字作为参数传递给`JSONRpc`对象

下面是一个RPC客户端，主动`connect`PRC服务器，然后保存`self.server`用作调用远程方法

因为是TCP连接，在客户端调用`close()`方法之前，服务器也可通过RPC调用客户端服务

```
import socket
from bsonrpc import JSONRpc, request, service_class, notification

@service_class
class MyRpcServer(object):

    @request
    def my_double(self, a):
        return a*2

    @notification
    def report_progress(self, mes):
        print mes

class ClientServer(object):

    def __init__(self):
        self.s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        self.s.connect(('localhost', 6000))
        self.rpc = JSONRpc(self.s, MyRpcServer())
        self.server = self.rpc.get_peer_proxy()

    def add(self, a, b):
        return self.server.my_progress_add(a, b)

    def close(self):
        self.rpc.close()

c = ClientServer()
print c.add(2, 2)
```

服务端和客户端类似，不同的是服务器需要等待客户端服务，因此使用线程会比较方便

通过使用单例模式，程序的其他地方可以通过实例化一个RPC类获取`client`，用来调用客户端方法

```
import socket
from bsonrpc import service_class, request, JSONRpc, rpc_request
import threading
import time


@service_class
class MyRpcServer(object):

    def __new__(cls):
        """单例模式"""
        if "_sin" not in cls.__dict__:
            cls._sin = super(MyRpcServer, cls).__new__(cls)
        return cls._sin

    def __init__(self):
        if 'client' not in self.__dict__:
            self.client = None

    @request
    def my_add(self, num_1, num_2):
        return num_1 + num_2

    @rpc_request
    def my_progress_add(self, rpc, num_1, num_2):
        print rpc.client_info
        self.client = rpc.get_peer_proxy()
        self.client.n.report_progress('hi')
        result = self.client.my_double(3)
        print result
        return num_1 + num_2


class ServiceServer(threading.Thread):

    def __init__(self):
        self.ss = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        self.ss.bind(('localhost', 6000))
        self.ss.listen(10)

        threading.Thread.__init__(self)

    def run(self):
        while True:
            _s, _address = self.ss.accept()
            JSONRpc(_s,
                    MyRpcServer(),
                    client_info=_address)

s = ServiceServer()
s.start()
time.sleep(5)
c = MyRpcServer()
res = c.client.my_double(5)
print id(c), res
b = MyRpcServer()
res = b.client.my_double(10)
print id(b), res
```

其实JSONRpc()函数返回的就是一个rpc对象，可以在线程中保存，以便获得client对象

```
class ServiceServer(threading.Thread):

    def __init__(self):
        self.ss = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        self.ss.bind(('localhost', 6000))
        self.ss.listen(10)
        self.rpc = None

        threading.Thread.__init__(self)

    def run(self):
        while True:
            _s, _address = self.ss.accept()
            self.rpc = JSONRpc(_s,
                               MyRpcServer(),
                               client_info=_address)
            print "name", _s.getsockname()
```

然后这样获得client

```
s = ServiceServer()
s.start()
time.sleep(5)
client = s.rpc.get_peer_proxy()
print client.my_double(5)  # 调用客户端方法
```

试着用循环处理多个客户端（并没有这个必要）

```
class ServiceServer(threading.Thread):

    def __init__(self):
        self.ss = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        self.ss.bind(('localhost', 6000))
        self.ss.listen(10)
        self.rpcs = []

        threading.Thread.__init__(self)

    def run(self):
        while True:
            _s, _address = self.ss.accept()
            self.rpcs.append(JSONRpc(_s,
                                     MyRpcServer(),
                                     client_info=_address))

s = ServiceServer()
s.start()
while True:
    time.sleep(5)
    for rpc in s.rpcs[:]:
        try:
            client = rpc.get_peer_proxy()
            print client.my_double(5)
        except BsonRpcError:
            rpc.close()
            s.rpcs.remove(rpc)
```
