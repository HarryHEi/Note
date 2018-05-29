---
title: 任务式协程
date: 2018-05-29 16:48:00
tags: [python]
---

参考：[A Curious Course on Coroutines and Concurrency](http://www.dabeaz.com/coroutines/Coroutines.pdf)

```
from queue import Queue
import socket
import select
import types


class Task(object):
    """
    任务类
    """
    task_id = 0

    def __init__(self, target):
        Task.task_id += 1
        self.tid = Task.task_id  # 任务标识
        self.target = target  # 任务协程
        self.send_val = None  # 发送的值
        self.stack = []  # 调用堆栈

    def run(self):
        while True:
            try:
                # 执行到下一个yield
                result = self.target.send(self.send_val)
                # 如果是系统调用，返回给调度器处理
                if isinstance(result, SystemCall):
                    return result
                # 如果返回的是生成器，表示要进行跳跃
                if isinstance(result, types.GeneratorType):
                    self.stack.append(self.target)
                    self.send_val = None
                    self.target = result
                else:
                    # 如果stack为空，表示执行完成
                    if not self.stack:
                        return
                    # 从栈中取出一个生成器（协程），将result作为值发送给迭代器
                    self.send_val = result
                    self.target = self.stack.pop()
            except StopIteration:
                # 处理结束的生成器，从栈取一个生成器(协程)运行
                if not self.stack:
                    raise
                self.send_val = None
                self.target = self.stack.pop()




class SystemCall(object):
    """系统调用基类，所有系统调用操作需要继承这个类实现"""
    def __init__(self):
        self.task = None
        self.schedule = None

    def handle(self):
        pass


class Scheduler(object):
    """调度类"""
    def __init__(self):
        self.ready = Queue()  # 用来存放即将运行的Task
        self.task_map = {}  # task_id => task
        self.exit_waiting = {}  # 存放正在等待的Task实例，task_id => [tasks]
        self.read_waiting = {}  # 等待read的sockets， fd => task
        self.write_waiting = {}  # 等待write的sockets

    def schedule(self, task):
        """将一个任务放到准备队列"""
        self.ready.put(task)

    def new(self, target):
        """新增一个任务"""
        new_task = Task(target)
        self.task_map[new_task.tid] = new_task
        self.schedule(new_task)
        return new_task.tid

    def exit(self, task):
        """移除一个任务"""
        print('Task {} terminated'.format(task.tid))
        del self.task_map[task.tid]
        # 当一个Task停止时，检查是否有Task在等待这个Task执行结束，并将等待的Task加入运行队列，继续执行
        # 一个Task可能被很多Task等待
        for task in self.exit_waiting.pop(task.tid, []):
            self.schedule(task)

    def wait_for_exit(self, task, wait_tid):
        """使得一个task等待wait_tid对应的Task"""
        if wait_tid in self.task_map:
            # 将Task追加到exit_waiting中wait_tid对应的列表后面
            self.exit_waiting.setdefault(wait_tid, []).append(task)
            return True
        else:
            # 如果等待的Task不在这个调度器，返回False
            return False

    def wait_for_read(self, task, fd):
        """将task和文件描述符保存到read_waiting字典"""
        self.read_waiting[fd] = task

    def wait_for_write(self, task, fd):
        self.write_waiting[fd] = task

    def io_poll(self, timeout):
        """轮询I/O，使用select监听使用的套接字"""
        if self.read_waiting or self.write_waiting:
            r, w, e = select.select(self.read_waiting, self.write_waiting, [], timeout)
            for fd in r:
                self.schedule(self.read_waiting.pop(fd))
            for fd in w:
                self.schedule(self.write_waiting.pop(fd))

    def io_task(self):
        """轮询I/O的Task"""
        while True:
            if self.ready.empty():
                self.io_poll(None)
            else:
                self.io_poll(0)
            yield

    def main_loop(self):
        """调度器主循环，从准备队列取出任务，并运行到下一个yield"""
        self.new(self.io_task())  # 轮询I/O
        while self.task_map:
            task = self.ready.get()
            try:
                result = task.run()
                # 如果Task yield的值是一个系统调用，代表这个任务进行系统调用
                if isinstance(result, SystemCall):
                    result.task = task  # 保存任务实例
                    result.schedule = self  # 保存调度器实例
                    result.handle()
                    continue
            except StopIteration:
                self.exit(task)
                continue
            self.schedule(task)


class GetTid(SystemCall):
    """一个返回任务ID的系统调用"""
    def handle(self):
        # 获取TID，赋值给Task实例的send_val，这个值会通过send方法发送给yield语句
        self.task.send_val = self.task.tid
        self.schedule.schedule(self.task)


class NewTask(SystemCall):
    """新建一个任务的系统调用"""
    def __init__(self, target):
        """传入一个(生成器)协程"""
        super(NewTask, self).__init__()

        self.target = target

    def handle(self):
        # 操作调度器新增一个任务
        tid = self.schedule.new(self.target)
        # Task的TID作为yield返回值
        self.task.send_val = tid
        self.schedule.schedule(self.task)


class KillTask(SystemCall):
    """删除一个Task的系统调用"""
    def __init__(self, tid):
        """传入tid参数"""
        super(KillTask, self).__init__()

        self.tid = tid

    def handle(self):
        # 从调度器的task_map中取出一个Task实例
        task = self.schedule.task_map.get(self.tid)
        if task:
            # 将Task实例中的（生成器）协程关闭
            task.target.close()
            # 向yield发送一个True表示关闭成功
            self.task.send_val = True
        else:
            # 向yield发送一个False表示关闭失败(Task不存在)
            self.task.send_val = False
        # 立刻将这个Task加入执行队列，将抛出StopIteration异常而终止
        self.schedule.schedule(self.task)


class WaitTask(SystemCall):
    """等待另外一个Task的系统调用"""
    def __init__(self, tid):
        """传入需要等待的Task的tid"""
        super(WaitTask, self).__init__()

        self.tid = tid

    def handle(self):
        # 通过调度器使一个Task等待另一个Task退出
        result = self.schedule.wait_for_exit(self.task, self.tid)
        self.task.send_val = result
        if not result:
            # 如果返回False，可能Task已经结束，Task添加到执行列表，继续执行
            self.schedule.schedule(self.task)


class ReadWait(SystemCall):
    """等待read的系统调用 """
    def __init__(self, sk):
        super(ReadWait, self).__init__()

        self.sk = sk

    def handle(self):
        fd = self.sk.fileno()  # 从套接字获取文件描述符
        self.schedule.wait_for_read(self.task, fd)  # 添加到read_waiting，使用select监听I/O


class WriteWait(SystemCall):
    def __init__(self, sk):
        super(WriteWait, self).__init__()

        self.sk = sk

    def handle(self):
        fd = self.sk.fileno()
        self.schedule.wait_for_write(self.task, fd)


class Socket(object):
    """一个异步套接字"""
    def __init__(self, sock):
        self.sock = sock

    def accept(self):
        # 等待服务端读事件
        yield ReadWait(self.sock)
        client, address = self.sock.accept()
        # yield一个客户端Socket和地址
        yield Socket(client), address

    def send(self, data):
        while data:
            # 等待写事件
            yield WriteWait(self.sock)
            # 逐个写完
            len = self.sock.send(data)
            data = data[len:]

    def recv(self, max_bytes):
        # 等待读事件
        yield ReadWait(self.sock)
        # 收数据
        yield self.sock.recv(max_bytes)

    def close(self):
        # 关闭套接字
        yield self.sock.close()


def handle_client(client, address):
    """处理客户端的连接"""
    print('new connection', address)
    while True:
        data = yield client.recv(1024)
        if not data:
            break
        yield client.send(data)
    yield client.close()
    print('client closed')
    yield  # 纯粹是为了使这个函数变为生成器函数


def server():
    print('server start')
    sk = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    sk.bind(('0.0.0.0', 5678))
    sk.listen(5)
    # 把套接字包装成异步套接字
    sock = Socket(sk)
    while True:
        # 等待连接，运行一个新的Task处理客户端连接
        client, address = yield sock.accept()
        yield NewTask(handle_client(client, address))


schedule = Scheduler()
schedule.new(server())
schedule.main_loop()
```
