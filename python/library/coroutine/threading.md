---
title: 多线程
date: 2016-03-19 20:35:43
tags: [python]
---

# 什么时进程

进程(有时被称为**重量级进程**)是程序的一次执行。
**每个进程都有自己的地址空间,内存,数据栈以及其它记录其运行轨迹的辅助数据。**
操作系统管理在其上运行的所有进程,并为这些进程公平地分配时间。
进程也可以通过 fork 和 spawn 操作来完成其它的任务。
不过各个进程有自己的内存空间,数据栈等,所以只能使用进程间通讯(IPC),
而不能直接共享信息。

**Python 实现多进程**
Unix/Linux提供一个**fork()**函数调用
**fork()**函数把当前进程(父进程)复制了一份(子进程)，分别在父进程和子进程返回一个值
子进程永远返回0，父进程返回进程的ID，子进程可以通过**getppid()**拿到父进程PID

```
import os

pid = os.fork()

if pid == 0:
    print "my pid is ",os.getppid()
else:
    print "my pid is ",pid
```

输出结果

```
my pid is  19009
my pid is  1950
```

# 什么是线程

线程(有时被称为**轻量级进程**)跟进程有些相似,不同的是,所有的线程运行在同一个进程中,
**共享相同的运行环境。**
它们可以想像成是在主进程或“主线程”中并行运行的“迷你进程”。
线程有开始,顺序执行和结束三部分。
它有一个自己的指令指针,记录自己运行到什么地方。
线程的运行可能被抢占(中断),或暂时的被挂起(也叫睡眠),让其它的线程运行,这叫做让步。
**一个进程中的各个线程之间共享同一片数据空间,所以线程之间可以比进程之间更方便地共享数据以及相互通讯。**
线程一般都是并发执行的,正是由于这种并行和数据共享的机制使得多个任务的合作变为可能。
实际上,在单 CPU 的系统中,真正的并发是不可能的,每个线程会被安排成每次只运行一小会,
然后就把 CPU 让出来,让其它的线程去运行。
在进程的整个运行过程中,每个线程都只做自己的事,在需要的时候跟其它的线程共享运行的结果。
当然,这样的共享并不是完全没有危险的。如果多个线程共同访问同一片数据,则由于数据访
问的顺序不一样,有可能导致数据结果的不一致的问题。这叫做竞态条件(race condition)。
幸运的是,大多数线程库都带有一系列的同步原语,来控制线程的执行和数据的访问。
另一个要注意的地方是,由于有的函数会在完成之前阻塞住,在没有特别为多线程做修改的情
况下,这种“贪婪”的函数会让 CPU 的时间分配有所倾斜。
导致各个线程分配到的运行时间可能不尽相同,不尽公平。

# 计算密集型程序和I/O密集型程序
涉及到网络、磁盘IO的任务都是IO密集型任务，这类任务的特点是CPU消耗很少
任务的大部分时间都在等待IO操作完成（因为IO的速度远远低于CPU和内存的速度）。
**对于IO密集型任务，任务越多，CPU效率越高**，但也有一个限度。
常见的大部分任务都是IO密集型任务，比如Web应用。
**IO密集型任务执行期间，99%的时间都花在IO上，花在CPU上的时间很少。**
因此，用运行速度极快的C语言替换用Python这样运行速度极低的脚本语言，完全无法提升运行效率。
对于IO密集型任务，最合适的语言就是开发效率最高（代码量最少）的语言，脚本语言是首选。

计算密集型任务的特点是要进行**大量的计算**，消耗CPU资源，比如计算圆周率、对视频进行高清解码等等，全靠CPU的运算能力。
这种计算密集型任务虽然也可以用多任务完成，但是任务越多，花在任务切换的时间就越多，CPU执行任务的效率就越低。
所以，要最高效地利用CPU，计算密集型任务同时进行的数量应当等于CPU的核心数。
Python这样的脚本语言运行效率很低，完全不适合计算密集型任务。

# python 多线程
Python 代码的执行由 Python 虚拟机(也叫解释器主循环)来控制。
Python 在设计之初就考虑到要在主循环中,同时只有一个线程在执行,
就像单 CPU 的系统中运行多个进程那样,内存中可以存放多个程序,
但任意时刻,只有一个程序在 CPU 中运行。
同样地,虽然 Python 解释器中可以“运行”多个线程,但在任意时刻,只有一个线程在解释器中运行。

python中使用多线程有两种方式，分别是使用函数传递和类包装。
python标准库提供了thread和threading两个模块。

> 这两个模块中,我们不建议使用 thread模块。
这样做有很多原因,很明显的一个原因是,当主线程退出的时候,所有其它线程没有被清除
就退出了。但另一个模块 threading 就能确保所有“重要的”子线程都退出后,进程才会结束。

---

thread 模块提为线程提供了一个低级接口

thread.start_new_thread(function, args[,kwargs])

创建一个新线程，第二个参数必须是一个元祖，如果只有没有参数
或者只有一个，要在括号里加一个‘，’
函数返回值是一个整数标识符

thread.exit()

结束当前线程，可以在函数中调用，以关闭进程
当主线程退出的时候, 所有的线程也随着退出

thread.get_ident()

获取当前线程的标识符

thread.interrupt_main ()

在主线程触发 KeyboardInterrupt 异常，用于中断线程

```
import thread, time
thread.start_new_thread(lambda : (thread.interrupt_main(), ), ())
try:
    time.sleep(2)
except KeyboardInterrupt, e:
    print 'error:', e
print 'over'
```

---

thread锁

thread.allocate_lock()

创建锁

lock.acquire(waitflag=None)

获取锁

lock.release()

释放锁

---

threading

threading是thread模块的二次封装版本
threading 模块为线程提供了一个高级接口

threading 模块支持守护线程,它们是这样工作的:
守护线程一般是一个等待客户请求的服务器,如果没有客户提出请求,它就在那等着。
如果主线程要退出的时候,不用等待那些子线程完成,那就设定这些线程的 daemon 属性。
即,**在线程开始(调用 thread.start())之前,调用 setDaemon()函数设定线程的 daemon 标志
thread.setDaemon(True)就表示这个线程“不重要”
如果你想要等待子线程完成再退出,那就什么都不用做,或者显式地调用
thread.setDaemon(False)以保证其 daemon 标志为 False。**
可以调用 thread.isDaemon()函数来判断其 daemon 标志的值。
新的子线程会继承其父线程的 daemon 标志。整个 Python 会在所有的非守护
线程退出后才会结束,即进程中没有非守护线程存在的时候才结束。

threading.Thread()

```
thread = threading.Thread(target=function,args=(...))
```

继承Thread类，定义run方法，就可以创建一个新线程
然后调用start方法

```
import threading
import time

class Counter:
    def __init__(self):
        self.lock = threading.Lock()
        self.value = 0

    def increment(self):
        self.lock.acquire() #获取锁
        self.value += 1
        self.lock.release() #释放锁
        return self.value

count = Counter()

class MyThread(threading.Thread):
    def run(self):
        for i in range(10):
            value = count.increment()  # 自增
            time.sleep(1)
            print self.getName(),"value: %d"%value

# 依次开启十个线程
for i in range(10):
    MyThread().start()
```

类初始化模型
def __init__(self, group=None, target=None, name=None, args=(), kwargs{})

+ group: 预留
+ target: 可调用函数
+ name: 线程名
+ args,kwargs: 参数

```
#!/usr/bin/env python
# -*-coding:utf-8 -*-

import threading
from time import sleep, ctime

class MyThread(threading.Thread):
    def  __init__(self, target, args, name=''):
        super(MyThread, self).__init__()
        self.target = target
        self.args = args
        self.name = name

    def run(self):
        print 'starting', self.name, 'at:', ctime()
        self.res = apply(self.target, self.args)
        print self.name, 'finished at:', ctime()

    def get_res(self):
        return self.res

def loop(time):
    sleep(time)

thread = MyThread(loop, (2,))
thread.start()
```

threading.Thread()的方法

```
thread.getName() 获取线程名称

thread.ident() 获取线程标识符

thread.is_alive()
thread.isAlive()
判断线程是否是激活的。从调用start()方法启动线程，到run()方法执行完毕或者
遇到未处理异常而中断,这段时间线程是激活的。

thread.join([timeout])
调用join会使主调线路阻塞，直到该线程运行结束或超时，参数timeout是数值类
型，表示时间，单位秒
```

举例

```
#!/usr/bin/env python
# -*-coding:utf-8 -*-

import threading
from time import sleep, ctime
loops = [4,2]
def loop(nloop, nsec):
    print 'start loop', nloop, 'at:', ctime()
    sleep(nsec)
    print 'loop', nloop, 'done at:', ctime()
    
def main():
    print 'starting at:', ctime()
    threads = []
    nloops = range(len(loops))

    for i in nloops: #创建线程
        thread = threading.Thread(target=loop,args=(i, loops[i]))
        threads.append(thread)
    
    for thread in threads: #开始所有线程
        thread.start()
    
    for thread in threads: #阻塞等待
        thread.join()
    
    print 'all DONE at:', ctime()
    
if __name__ == '__main__':
    main()
```

终止线程 自定义线程类

循环检测Event，然后终止循环
重写join方法，在join方法内设置Event

```
import threading
import time

class myThread(threading.Thread):
    def __init__(self, name="myThread"):
        threading.Thread.__init__(self, name=name)
        self._stopevent = threading.Event()
    def run(self):
        while not self._stopevent.isSet():
            print "loop"
            self._stopevent.wait(1.0)
        print "ends"
    def join(self, timeout=None):
        self._stopevent.set()
        threading.Thread.join(self, timeout)

thread = myThread()
thread.start()
time.sleep(5) #等待5秒
thread.join() #终止线程
```

---

threading模块有threading.Lock和threading.RLock

RLock允许同一线程中多次获取锁，但是对应着也要多次release，Lock不能在同一线程多次获取锁

使用Lock

```
import threading
import time


class A(threading.Thread):
    def __init__(self, lock):
        super(A, self).__init__()
        self.lock = lock

    def run(self):
        self.lock.acquire()
        time.sleep(1)
        print "ok"
        self.lock.release()

a_lock = threading.Lock()
a1 = A(a_lock)
a2 = A(a_lock)
a3 = A(a_lock)

a1.start()
a2.start()
a3.start()
```

因为锁的存在，每隔一秒输出一个ok

在同一个线程获取多次

```
import threading

lock = threading.Lock()
lock.acquire()
lock.acquire()  # 锁死了
lock.release()
lock.release()
```

使用RLock，正常。

```
import threading

lock = threading.Lock()
lock.acquire()
lock.acquire()
lock.release()
lock.release()  # 需要多次release，与acquire一一对应
```

---

threading.Condition()

条件同步机制，一个线程等待特定条件，另一个线程发出特定条件满足的信号，可以传入一个锁参数，默认为一个RLock

condition.wait([timeout])

释放占用的锁，挂起线程，直到被唤醒或者超时

condition.nodify()

唤醒一个被挂起的线程，不会释放锁

condition.notify_all()

condition.notifyAll()

唤醒所有被挂起的线程

用poster和geter之间的对话模拟

```
# -*- coding: utf-8 -*-  
import threading
import time
import random


class Poster(threading.Thread):
    def __init__(self, cond, name):
        super(Poster, self).__init__()
        self.cond = cond
        self.name = name

    def run(self):
        self.cond.acquire()
        print "我要登录~"
        self.cond.wait()
        time.sleep(random.randint(1, 5))
        print "给你一个数据1,你还要啥?"
        self.cond.notify()
        self.cond.wait()
        time.sleep(random.randint(1, 5))
        print "给你一个数据2,好了没?"
        self.cond.notify()
        self.cond.wait()
        self.cond.release()
        time.sleep(random.randint(1, 5))
        print "幸苦了"


class Getter(threading.Thread):
    def __init__(self, cond, name):
        super(Getter, self).__init__()
        self.cond = cond
        self.name = name

    def run(self):
        self.cond.acquire()
        time.sleep(random.randint(1, 5))
        print "好的，我要一个数据1!"
        self.cond.notify()
        self.cond.wait()
        time.sleep(random.randint(1, 5))
        print "我还要一个数据2~"
        self.cond.notify()
        self.cond.wait()
        time.sleep(random.randint(1, 5))
        print "好了！"
        self.cond.notify()
        self.cond.release()
        time.sleep(random.randint(1, 5))
        print "多谢合作"

c = threading.Condition(lock)
poster = Poster(c, "poster")
getter = Getter(c, "getter")
poster.start()
getter.start()
```

输出

```
我要登录~
好的，我要一个数据1!
给你一个数据1,你还要啥?
我还要一个数据2~
给你一个数据2,好了没?
好了！
多谢合作
幸苦了
```

参考
<http://python.jobbole.com/81544/>
<http://python.jobbole.com/81546/>
