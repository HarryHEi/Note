---
title: asio
date: 2018-02-28 15:00:35
tags: [c_plus]
---


[asio](http://think-async.com)

asio具有non-boost和boost版本。

当使用C++11编译器时，如果不使用Boost库，添加ASIO_STANDALONE参数。
cmake添加`add_definitions(-DASIO_STANDALONE)`。

同步调用步骤如下:

1. 首先需要有一个io_service

    ```
    asio::io_service io_service;
    ```

2. 初始化并连接IO设备;

    ```
    asio::ip::tcp::socket socket(io_service);
    socket.connect(endpoint);
    ```

3. IO设备传递请求给io_service;
4. io_service通过操作系统调用;
5. 操作系统返回结果给io_service;
6. io_service把返回的错误用"asio::error_code"表示(通常0表示没有错误)，调用结果返回给IO设备。
7. 如果调用失败，IO设备抛出一个"asio::system_error"异常。

    如果调用的时候传递一个error_code参数，错误将被保存，不会抛出异常
    ```
    asio::error_code ec;
    socket.connect(endpoint, ec);
    ```

异步调用稍有不同:

1. 初始化并连接IO设备;

    ```
    socket.async_connect(endpoint, handler);
    ```

    这里handler是一个函数或者函数对象，格式如下:
    ```
    void handler(const asio::error_code& ec);
    ```

2. IO对象传递请求给io_service;
3. io_service发信号给操作系统进行异步连接。

    等待一段时间，如果是同步操作，这段时将包括在连接的时间内。

4. 操作系统将结果放在队列中表明连接操作完成，并准备传递给io_service。
5. 程序必须调用io_service::run()或类似的一个成员函数，当有未完成的异步操作时，该调用阻塞，所以可以在开始异步操作之后调用。
6. 在io_service::run()运行时，io_service将结果出队并表示为error_code，然后传递给handler函数。

# proactor设计模式

没有线程的并发。

asio的异步支持基于proactor设计模式。

![](http://think-async.com/Asio/asio-1.10.6/doc/proactor.png)

Asynchronous Operation(异步操作)
定义异步执行的操作，例如在套接字上的读取和写入。

Asynchronous Operation Processor(异步操作处理器)
执行Asynchronous Operation，并在操作完成时将Completion Event Queue上的事件队列化。

Completion Event Queue(完成事件队列)
缓存完成事件，直到被Asynchronous Event Demultiplexer出列。

Completion Handler(完成处理程序)
处理Asynchronous Operation的结果，一般是函数对象。

Asynchronous Event Demultiplexer(异步事件解复用器)
阻塞等待Completion Event Queue上的事件，然后返回一个完成事件给调用者。

Proactor
调用Asynchronous Event Demultiplexer使事件出列，并调度与事件相关联的Completion Handler，由io_service表示。

Initiator
应用程序相关的代码，用于启动Asynchronous Operation，Initiator在比较高的级别接口与Asynchronous Operation Processor交互。

## 使用Reactor实现
在许多平台，Proactor由Reactor实现，比如select、epoll或者kqueue，实现方法对应Proactor如下:

Asynchronous Operation Processor
首先有一个使用select、epoll或者kqueue实现的Reactor，当Reactor准备好执行操作时，Processor执行Asynchronous Opration，并且把相关的Completion Handler列入Completion Event Queue。

Completion Event Queue
由多个Competion Handler构成的列表。

Asynchronous Event Demultiplexer
等待事件或者条件变量直到Completion Event Queue有Completion Handler可以操作。
