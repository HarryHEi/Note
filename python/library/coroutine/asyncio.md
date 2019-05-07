---
title: asyncio
date: 2019-05-07 14:31:00
tags: [python]
---

[Python 3 – An Intro to asyncio](https://www.blog.pythonlibrary.org/2016/07/26/python-3-an-intro-to-asyncio/)
[I don't understand Python's Asyncio](http://lucumr.pocoo.org/2016/10/30/i-dont-understand-asyncio/)

[asyncio](https://docs.python.org/3/library/asyncio.html)是在Python 3.4版本添加，提供单线程实现并发的方案，不向前兼容，旧版本可以通过`Twisted`或者`gevent`实现单线程并发。

# 定义
asyncio是一个提供事件循环的框架，时间循环会等待并执行事件，可以处理IO和系统事件

## Coroutines
即协程方法(coroutine function)，协程方法是由关键字`async`和`await`定义的（关键字在Python 3.5添加）方法，旧版本可以通过`@asyncio.coroutine`装饰器和`yield from`定义。

直接调用协程只会返回一个协程对象(coroutine object)，通常有下面几种执行方式：
+ 通过`asyncio.run()`方法（在Python 3.7版本添加），该方法会创建一个事件循环，然后执行该协程，结束后关闭事件循环。
+ 在协程中通过`await`关键字执行。
+ 通过`asyncio.create_task()`方法（在Python 3.7版本添加）把协程包装成`Task`对象并执行，返回该任务(task)，任务会在当前在运行的事件循环中执行(通过`get_running_loop()` 返回的事件循环)，如果没有事件循环在运行，会抛出异常。

`async`和`await`可以被视作实现异步变成的API。asyncio是个用`async`和`await`异步编程的框架。

## Awaitables
可异步调用(awaitable)对象指的是可以使用`await`表达式调用的对象。asyncio中的awaitable对象主要为：协程(coroutines)， 任务(Tasks)和Future。

### Coroutines 
协程（包括协程方法和协程对象）是可异步调用对象，在一个协程中可以通过`await`异步调用其他协程。

### Tasks
任务是Future的子类，用于包装协程。任务用于并发的执行协程，当协程使用`asyncio.create_task()`包装成Task时，协程会被执行，通过`await`关键字调用任务，会等待任务执行直到任务执行完成。

### Futures
Future是特殊的底层的可异步调用对象，`await`会等待Future对象获得结果，一般情况下应用代码不会主动创建Future，可能来自其他框架或者API，比如`loop.run_in_executor()`。
