---
title: 常见服务模型
date: 2018-3-3 21:55:33
tags: [basic]
---


# 多进程服务端

缺点：

1. 创建进程需要大量的运算和内存空间。
2. 数据交换方法比较复杂，需要使用特殊的IPC技术。

伪代码
```
server_sock = socket()
server_sock.bind(endpoint)
server_sock.listen()

while(1)
{
    clt_sock = server_sock.accept() //等待客户连接

    pid = fork() //有客户连接后创建新的进程
    
    if (pid == 0) //表示子进程，处理客户数据。
    {
        server_sock.close()
        while (data = clt_sock.read())
            write(data)

        clt_sock.close()
        return 0
    }
    else //父进程关闭套接字
        clt_sock.close()
}
```

# I/O复用服务器之select

使用select函数实现服用服务器。

优点：

大部分操作系统都支持select，兼容性好。

缺点：

1. 每次调用select都需要传入监视对象的信息。
2. 调用完select之后需要遍历文件描述符。
3. 创建进程时的上下文切换造成的巨大开销。

伪代码
```
server_sock = socket()
server_sock.bind(endpoint)
server_sock.listen()

event_lis.add(server_sock) //事件中监听服务端套接字变化

while(1)
{
    cyp_event_lis = event_lis  //事件列表赋值
    fd = select(cyp_event_lis)  //在循环中调用select监听

    for(i=0; i < max; ++i)  //遍历所有描述符
    {
        if (fd == server_sock) //服务器套接字变化，表示有新的连接。
        {
            clt_sock = server_sock.accept()
            event_lis.add(clt_sock)
        }
        else //客户端套接字变化，表示有新的数据。
        {
            data = i.read()
            if (len == 0)
                i.close()
            else
                write(data)
        }
    }
}
```

# linux I/O复用服务器之epoll

epoll是改进的I/O复用模型，只支持linux系统。

优点：

1. 无需编写以监视状态变化为目的所有文件描述符的循环语句。
2. 调用对应epoll函数时，无需每次传入监视对象信息。

伪代码
```
server_sock = socket()
server_sock.bind(endpoint)
server_sock.listen()

epoll_fd = epoll_create()  //创建epoll资源
epoll_fd.add(server_sock)  //事件中监听服务端套接字变化
events = []  //保存发生事件的包含文件描述符结构体

while(1)
{
    event_count = epoll_wait(epoll_fd, events)  //监听事件，返回事件数

    for (i=0; i < event_count; ++i)  //根据事件数遍历
    {
        if (events[i].fd == server_sock)  //服务器套接字变化，有新连接。
        {
            clt_sock = server_sock.accept()
            event_lis.add(clt_sock)
        }
        else
        {
            data = events[i].fd.read()
            if (len == 0)
                events[i].fd.close()
            else
                write(data)
        }
    }
}
```

# 多线程I/O模型

优点：

1. 线程比进程的创建和上下文切换更快。
2. 线程间交换数据时无需特殊技术。

伪代码
```
server_sock = socket()
server_sock.bind(endpoint)
server_sock.listen()

clt_socks = []

while(1)
{
    clt_sock = server_sock.accept()  //等待客户端连接

    lock()  //加锁
    clk_socks.add(clt_sock)  //添加客户端套接字
    unlock()  //解锁

    t = pthread_create(handler)
    t.detach()
}

void *handler(void *arg)
{
    clt_sock = do_cast<sock>(arg)

    while (data = clt_sock.read())
        write(data)

    clt_sock.close()

    lock()
    clk_socks.erase(clt_sock)
    unlock()

    return NULL
}
```

# windwos 异步通知I/O模型

伪代码
```
socks = []  //用于存放套接字
events = []  //用于存放事件

server_sock = socket()
server_sock.bind(endpoint)
server_sock.listen()

new_event = event_create()  //创建事件对象
select(server_sock, new_event)  //绑定服务端套接字，会直接返回。

socks.add(server_sock)  //套接字添加到数组
events.add(new_event)  //事件添加到数组

while(1)
{
    event = wait_for_events(socks)  //阻塞等待事件

    for (i=0; i < socks.size(); ++i)  //根据索引依次验证
    {
        event_type = event_clear(socks[i], events[i])  //复位状态，返回事件类型。

        if (event_type.has(ACCEPT))  //验证是否是有accept事件
        {
            clt_sock = server_sock.accept()  //获得客户端套接字
            new_event = event_create()  //新建事件对象
            select(clt_sock, new_event)  //绑定客户端套接字

            socks.add(server_sock)  //套接字添加到数组
            events.add(new_event)  //事件添加到数组
        }

        if (event_type.has(READ))  //验证是否有read事件
        {
            data = socks[i].read()
            write(data)
        }

        if(event_type.has(CLOSE))  //验证是否有close事件
        {
            event_close(events[i])
            socks[i].close()

            socks.erase(socks[i])
            events.erase(events[i])
        }
    }
}
```

# windows 重叠I/O模型

重叠I/O：同一线程内部向多个目标传输数据引起的I/O重叠现象。

伪代码
```
server_sock = socket(OVERLAPPED)  //创建一个重叠I/O套接字
server_sock.bind(endpoint)
server_sock.listen()

clt_sock = server_sock.accept()

event = event_create()
data = ''
server_sock.recv(OVERLAPPED, callback)

ret = WaitForEvents(event)  //阻塞等待事件
if (ret = WAIT_IO_COMPLETION)  //I/O完成
{
    puts('Overlapped I/O Completed')
}

void callback(rec_data)
{
    data = rec_data
}
```

# windows IOCP

Input Output Completion Port，输入输出完成端口

优点：

1. 非阻塞I/O，没有延迟。
2. 查找已完成的I/O时不需要循环。
3. 无需将套接字保存到数组管理。
4. 可以根据实验情况调整处理I/O的线程数。

伪代码：
```
com_port = comp_port_create()  //创建完成端口
for (i=0; i < number_processors; i++)  //创建多个监听线程
    create_thread(thread_main, com_port)

server_sock = socket(OVERLAPPED)  //创建一个重叠I/O套接字
server_sock.bind(endpoint)
server_sock.listen()

while(1)
{
    clt_sock = server_sock.accept()

    comp_port_create(clt_sock, com_port, th)  //套接字端口建立联系
    data = clt_sock.recv()  //接收
    write(data)
}


void thread_main(port)
{
    thread_com_port = port

    while(1)
    {
        info, size = get_comp_status(thread_com_port)  //获取完成端口状态
        sock = info.sock
        
        if (info.mode == READ)
        {
            if (data == 0)  //EOF关闭套接字
            {
                sock.close()
                continue
            }

            sock.send(some_message)  //发送

            data = sock.recv()  //接收
            write(data)
        }
        else
        {
            //表示发送事件完成
            puts('message sent')
        }
    }
}
```
