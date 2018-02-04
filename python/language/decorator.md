---
title: 装饰器
date: 2016-07-11 17:11:14
tags: [python]
---

# 装饰器

从语法角度讲，装饰器就是闭包的语法糖

dec接受一个函数参数，返回一个加工过的函数

```
#!/usr/bin/env python
#-*-coding:utf-8-*-

def dec(func):
    def in_dec(*arg):
        if len(arg) == 0:
            return 0
        return func(*arg)
    return in_dec

def my_sum(*arg):
    return sum(arg)

my_sum = dec(my_sum)


@dec
def add(a,b):
    return a+b

print my_sum(1,2)
print add(1,2)
```

输出

```
3
3
```

这样有个副作用，

使用了装饰器之后，函数发生了改变

```
print add.func_name
```

输出

```
in_dec
```

functools提供一个装饰器wraps消除这种副作用

```
def dec(func):
    @wraps(func)
    def in_dec(*arg):
        if len(arg) == 0:
            return 0
        return func(*arg)
    return in_dec
```

输出

```
add
```


# AOP

AOP是Aspect Oriented Programming（面向切面编程）的缩写，实现业务逻辑的各个部分分离。

比如说一些简单的打印函数

```
def say_hi(msg):
    print msg
```

目前函数功能是打印一个消息，后来需要添加一个功能，对消息进行过滤和谐。

可以直接修改函数本身，但是当类似这样的函数接口比较多或者函数比较复杂时，不易于维护。

这时候我们可以创建一个装饰器函数

```
def check(func):
    def wrapper(msg):
        if 'fuck' in msg:
            msg = msg.replace('fuck', '*')
            func(msg)
    return wrapper
```

这个装饰器函数的功能是接受一个函数作为参数，返回一个新的函数，该函数能过滤消息内容，并且调用原本的函数。

添加装饰器

```
@check
def say_hi(msg):
    print msg

say_hi("fuck you")
```

输出

```
* you
```

这样就在不修改原本函数的情况下实现了业务功能。