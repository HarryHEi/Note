---
title: RPC demo
date: 2018-05-29 14:15:00
tags: [python]
---

使用`bsonrpc`库和`gevent`库的RPC客户端和服务端示例

```
# -*- coding:utf-8 -*-

"""提供JSON-RPC服务

监听地址，等待连接
"""

import socket

import gevent.monkey
import gevent.server
import bsonrpc

from common.logging import logging

gevent.monkey.patch_socket()


class RpcManagerServer(gevent.Greenlet):
    """
    RPC 服务端

    >>> server = RpcManagerServer(('0.0.0.0', 6790))
    >>> server.start()
    >>> server.peer.setTunnelConfig(tunnelNo=0, frequency=10000)
    >>> server.rpc.invoke_request('setTunnelConfig', tunnelNo=0, frequency=10000)
    >>> server.join()

    如果需要提供RPC服务，可以自定义service_class，并把实例作为第二个参数传递

    可以指定call_back参数，当有客户端连接后会调用一次，这个可调用对象有一个参数，是该对象本身

    该类只会保存上一个连接的客户端的rpc对象，如果需要处理所有连接，需要编写保存RPC对象的代码
    """
    def __init__(self, address, server=None, call_back=None):
        """
        Args:
            address: tuple (ip, port)
        """
        super(RpcManagerServer, self).__init__()

        self.address = address
        self.server = server
        self.call_back = call_back
        self._sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        self._sock.bind(address)
        self._sock.listen(10)

        logging.info('RPC 监听地址 {}'.format(address))

        self.rpc = None
        self.peer = None

    def _run(self):
        while True:
            clt, address = self._sock.accept()
            logging.info('{}收到来自{}的新连接'.format(
                self.address,
                address
            ))
            self.rpc = bsonrpc.JSONRpc(
                clt,
                self.server,
                concurrent_notification_handling=bsonrpc.ThreadingModel.GEVENT,
                concurrent_request_handling=bsonrpc.ThreadingModel.GEVENT,
                threading_model=bsonrpc.ThreadingModel.GEVENT
            )
            self.peer = self.rpc.get_peer_proxy(timeout=5)
            if self.call_back:
                self.call_back(self)

```

```
# -*- coding:utf-8 -*-

"""连接终端的RPC服务

客户端类：
    主动连接客户端RPC，提供JSONRpc对象，用于进行调用。
"""

import socket

import gevent.monkey
import gevent.event
import bsonrpc

from common.logging import logging


gevent.monkey.patch_socket()


class RpcManagerClient(gevent.Greenlet):
    """RPC客户端

    >>> client = RpcManagerClient(('127.0.0.1', 6790))
    >>> client.start()
    >>> client.peer.setTunnelConfig(tunnelNo=0, frequency=10000)
    >>> client.rpc.invoke_request('setTunnelConfig', tunnelNo=0, frequency=10000)
    >>> client.close()

    如果需要提供RPC服务，可以自定义service_class，并把实例作为第二个参数传递

    可以指定call_back参数，当成功连接后会调用一次，这个可调用对象有一个参数，是该对象本身

    注意:
        close()需要在start()之后调用
        当调用close()之后不能再start()
    """

    def __init__(self, address, services=None, call_back=None):
        """
        :param address: (ip, port)
        :param services: server_class实例
        :param call_back: 一个可调用对象
        """
        super(RpcManagerClient, self).__init__()
        self.address = address
        self.services = services
        self.call_back = call_back
        self._sk = None
        self.rpc = None
        self.peer = None

        self._stop_event = gevent.event.Event()

    def _run(self):
        while not self._stop_event.is_set():
            try:
                logging.info('连接远程RPC {}'.format(self.address))
                self.peer = None
                self._sk = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
                self._sk.connect(self.address)
                self.rpc = bsonrpc.JSONRpc(
                    socket=self._sk,
                    services=self.services,
                    concurrent_notification_handling=bsonrpc.ThreadingModel.GEVENT,
                    concurrent_request_handling=bsonrpc.ThreadingModel.GEVENT,
                    threading_model=bsonrpc.ThreadingModel.GEVENT
                )
                self.peer = self.rpc.get_peer_proxy(timeout=5)
                if self.call_back:
                    self.call_back(self)
                self.rpc.join()
            except Exception as e:
                logging.error(e)
                logging.error('RPC{}连接失败'.format(self.address))
                self._stop_event.wait(5)

    def close(self):
        self._sk.close()
        self._stop_event.set()
        self.kill()

```
