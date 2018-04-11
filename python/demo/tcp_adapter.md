---
title: TCP数据中转
date: 2018-04-10 20:28:34
tags: [python]
---

用协程实现的tcp中转程序，把一个客户端的程序转发到其他客户端。

```
# -*- coding: utf-8 -*-

import gevent.monkey
from gevent.server import StreamServer
import gevent.queue
import gevent
import collections
import logging


root_logger = logging.getLogger('')
root_logger.setLevel(logging.INFO)
log_format = logging.Formatter('%(asctime)s.%(filename)s[line:%(lineno)d] %(levelname)s %(message)s')
stream_handler = logging.StreamHandler()
stream_handler.setFormatter(log_format)
root_logger.addHandler(stream_handler)


# 猴子补丁
gevent.monkey.patch_socket()

# 客户端类
Client = collections.namedtuple('Client', ['sock', 'address', 'queue'])

# 客户端容器，number -> Client
client_container = {}


class WriteLoop(gevent.Greenlet):
    """用于循环投递队列消息"""
    def __init__(self, sk, address, queue):
        super(WriteLoop, self).__init__()
        self.sk = sk
        self.address = address
        self.queue = queue

    def _run(self):
        while True:
            data = self.queue.get()
            for number, client in client_container.items():
                if client.sock != self.sk:
                    try:
                        client.sock.send(data)
                        logging.info('write {} from {} to {}'.format(data, self.address, client.address))
                    except Exception as e:
                        logging.error(e)
                        logging.error('{} disconnect'.format(self.address))


def number_generator():
    """用于生成递增数字"""
    start = 0
    while True:
        yield start
        start += 1

# 客户端标号生成器
number_gen = number_generator()


# 处理客户端连接
def handle(sock, address):
    logging.info('new connection, {}'.format(address))
    number = next(number_gen)  # 获取标识
    queue = gevent.queue.Queue()  # 初始化队列
    client_container[number] = Client(sock, address, queue)  # 创建客户端对象
    write_loop = WriteLoop(sock, address, queue)  # 队列的处理循环
    write_loop.start()
    while True:
        data = sock.recv(20)
        if not data:
            logging.info('{} disconnect'.format(address))
            break
        logging.info('{} put {}'.format(address, data))
        queue.put(data)
    # 释放该客户端资源
    write_loop.kill()
    write_loop.join()
    del client_container[number]
    logging.info('{} released'.format(address))


if __name__ == '__main__':
    logging.info('listen on 9999')
    server = StreamServer(('0.0.0.0', 9999), handle=handle)
    server.serve_forever()
```

或者扔到一个类里面
```
# -*- coding: utf-8 -*-

import gevent.monkey
from gevent.server import StreamServer
import gevent.queue
import gevent
import collections
import logging


root_logger = logging.getLogger('')
root_logger.setLevel(logging.INFO)
log_format = logging.Formatter('%(asctime)s.%(filename)s[line:%(lineno)d] %(levelname)s %(message)s')
stream_handler = logging.StreamHandler()
stream_handler.setFormatter(log_format)
root_logger.addHandler(stream_handler)


# 猴子补丁
gevent.monkey.patch_socket()

# 客户端类
Client = collections.namedtuple('Client', ['sock', 'address', 'queue'])


class WriteLoop(gevent.Greenlet):
    """用于循环投递队列消息"""
    def __init__(self, sk, address, queue, container):
        super(WriteLoop, self).__init__()
        self.sk = sk
        self.address = address
        self.queue = queue
        self.client_container = container

    def _run(self):
        while True:
            data = self.queue.get()
            for number, client in self.client_container.items():
                if client.sock != self.sk:
                    try:
                        client.sock.send(data)
                        # logging.info('write {} from {} to {}'.format(data, self.address, client.address))
                    except Exception as e:
                        logging.error(e)
                        logging.error('{} disconnect'.format(self.address))


def number_generator():
    """用于生成递增数字"""
    start = 0
    while True:
        yield start
        start += 1


class TCPAdapter(gevent.Greenlet):
    def __init__(self, address):
        super(TCPAdapter, self).__init__()
        self.client_container = {}
        self.number_gen = number_generator()
        self.server = StreamServer(address, handle=self._handle)

    def _handle(self, sock, address):
        logging.info('new connection, {}'.format(address))
        number = next(self.number_gen)  # 获取标识
        queue = gevent.queue.Queue()  # 初始化队列
        self.client_container[number] = Client(sock, address, queue)  # 创建客户端对象
        write_loop = WriteLoop(sock, address, queue, self.client_container)  # 队列的处理循环
        write_loop.start()
        while True:
            data = sock.recv(1024)
            if not data:
                logging.info('{} disconnect'.format(address))
                break
            # logging.info('{} put {}'.format(address, data))
            queue.put(data)
        # 释放该客户端资源
        write_loop.kill()
        write_loop.join()
        del self.client_container[number]
        logging.info('{} released'.format(address))

    def _run(self):
        self.server.serve_forever()


if __name__ == '__main__':
    address1 = ('0.0.0.0', 9001)
    address2 = ('0.0.0.0', 9002)
    logging.info('server on {}'.format(address1))
    logging.info('server on {}'.format(address2))
    adapter1 = TCPAdapter(address1)
    adapter2 = TCPAdapter(address2)
    adapter1.start()
    adapter2.start()
    gevent.joinall([adapter1, adapter2])

```
