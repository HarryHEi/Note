---
title: functools
date: 2018-1-27 20:10:37
tags: [python]
---

[文档](https://docs.python.org/3/library/functools.html)

functools模块用于高阶函数（表现为返回其他函数的函数）。

`function.reduce(function, iterable[, initializer])`
把接收两个参数的函数function用在iterable的每个元素上，每次的结果在和下一个元素累计调用。

```
>>> add = lambda a, b: a + b
>>> reduce(add, [1,2,3,4,5,6])
21
```

`functools.update_wrapper(wrapper, wrapped, asigned=WRAPPER_ASSIGNMENTS, updated=WRAPPER_UPDATES)`
可选参数是指定要分配给wrapper函数哪些原始函数的属性组成的元组，默认是`functools.WRAPPER_ASSIGNMENTS
`
```
>>> functools.WRAPPER_ASSIGNMENTS
('__module__', '__name__', '__qualname__', '__doc__', '__annotations__')
```
updated指定要更新的属性，默认`functools.WRAPPER_UPDATES`
```
>>> functools.WRAPPER_UPDATES
('__dict__',)
```
为了自省目的，该函数还会自动添加一个`__wrapped__`属性，标识被包装。

```
>>> def do_add(a, b):
...     return a + b
... 
>>> def wrapper(a, b):
...     print('do_add')
...     return do_add(a, b)
... 
>>> fun = wrapper
>>> fun.__name__
'wrapper'
>>> functools.update_wrapper(wrapper, do_add)
<function do_add at 0x000001C566346A60>
>>> fun = wrapper
>>> fun.__name__
'do_add'
```

`@functools.wraps(wrapped, assigned=WRAPPER_ASSIGNMENTS, updated=WRAPPER_UPDATES)`
`update_wrapper()`函数的装饰器语法糖。
```
def log(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        print('call {}'.format(func.__name__))
        return func(*args, **kwargs)
    return wrapper
```
针对带参数的三重包装
```
import functools

def log(text):  # 三重包装
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            print('{}:{}'.format(text, func.__name__))
            return func(*args, **kwargs)
        return wrapper
    return decorator


@log('do')
def now():
    print('1.28')


if __name__ == '__main__':
    n = now()
    print(now.__name__)  # now

```

`@functools.partial(func, *args, **keywords)`

用于把一个函数的一个参数固定，返回新的函数。（偏函数）
```
>>> int('1000', base=2)
8
>>> int('1000', base=10)
1000
>>> int_2 = functools.partial(int, base=2)
>>> int_2('1000')
8
```
