---
title: socket并发
date: 2017-03-24 17:27:28
tags: [soket]
---


服务器同时处理多个客户的并发服务器有许多种方式，可以使用Unix的fork函数创建子进程

(fork示例参考）(http://www.cnblogs.com/bastard/archive/2012/08/31/2664896.html)

本章内容使用的是python的多线程实现并发服务。


服务端使用SoketServer网络服务框架
如果设置daemon守护线程（t.daemon=True），在主线程结束时，子线程也会结束

```
import socket
import threading
import SocketServer


class TCPServerHandler(SocketServer.BaseRequestHandler):
    def handle(self):
        self.data = self.request.recv(1024).strip()
        print "{} wrote:".format(self.client_address[0])
        print self.data
        self.request.sendall(self.data.upper())


if __name__ == "__main__":
    host, port = "localhost", 6666
    server = SocketServer.TCPServer((host, port), TCPServerHandler)
    t = threading.Thread(target=server.serve_forever)
    #t.daemon = True
    t.start()

    # client
    sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

    try:
        sock.connect((host, port))
        sock.sendall("hello\n")
        rec = sock.recv(1024)
    finally:
        sock.close()

    print rec
```

SocketServer.TCPServer(server_address, RequestHandlerClass, bind_and_activate=true)使用TCP协议，提供客户端与服务器之间的连续数据流。当bind_and_activate设置为true（默认为true），构造函数会自动尝试调用server_bind() 绑定地址和server_activate() 监听地址

```
def __init__(self, server_address, RequestHandlerClass, bind_and_activate=True):
    """Constructor.  May be extended, do not override."""
    BaseServer.__init__(self, server_address, RequestHandlerClass)
    self.socket = socket.socket(self.address_family,
                                self.socket_type)
    if bind_and_activate:
        try:
            self.server_bind()
            self.server_activate()
        except:
            self.server_close()
            raise
```
