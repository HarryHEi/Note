# Greenlet 对象

Greenlet是一个轻量合作计划运行单元。

为了开启一个新的greenlet，需要传递目标函数以及它的参数给Grennlet构造器，然后start()，start()是非阻塞的，因此使用join()等待其运行结束:
```
def fun(x):
    print(x)

gl = gevent.Greenlet(run=fun, x=1)
gl.start()
gl.join()
```

或者使用快捷方式类方法spawn()，就不用调用start()了：
```
gl = gevent.Greenlet.spawn(fun, 1)
gl.join()
```

# 同步和异步执行

并发的核心思想是将很大的任务分解为一组子任务，这些任务可以同时异步执行，比如通过调用gevent.sleep(0)，在两个上下文之间切换：
```
def foo1():
    print(1)
    gevent.sleep(0)
    print(2)

def foo2():
    print(3)
    gevent.sleep(0)
    print(4)


gevent.joinall([
    gevent.spawn(foo1),
    gevent.spawn(foo2)
])
```
上例将打印1324。

# Monkey patching

Monkey patching将标准模块中的函数替换为支持合作功能的函数：
```
import socket
import gevent.monkey


gevent.monkey.patch_socket()

def sock_handler1(address):
    sk = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    sk.bind(address)
    sk.listen(10)
    while True:
        clt, address = sk.accept()
        while True:
            data = clt.recv(10)
            print(data)


def sock_handler2(address):
    sk = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    sk.bind(address)
    sk.listen(10)
    while True:
        clt, address = sk.accept()
        while True:
            data = clt.recv(10)
            print(data)


gevent.joinall([
    gevent.spawn(sock_handler1, ('0.0.0.0', 1234)),
    gevent.spawn(sock_handler1, ('0.0.0.0', 1235))
])
```
`gevent.monkey.patch_socket()`调用之后，socket就具有了合作并发的功能，这样能够同时处理来自两个地址的连接。
