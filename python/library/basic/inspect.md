---
title: inspect
date: 2019-6-11 17:41:00
tags: [python]
---

# inspect
[inspect](https://docs.python.org/3/library/inspect.html)提供查询对象信息的方法，比如查询对象类型、源码、方法参数等。

# 参数类型检查
利用`inspect`可以检查参数类型。

`inspect`提供`bind`方法，将位置信息和关键字信息和参数进行映射，如果`*args`和`**kwargs`和签名不一致，会抛出`TypeError`异常。

实例化一个签名
```
from inspect import Signature, Parameter


params = [
    Parameter('request', Parameter.POSITIONAL_OR_KEYWORD),
    Parameter('pk', Parameter.KEYWORD_ONLY, default=None)
]
sig = Signature(params)
print(sig)  # 输出 (request, *, pk=None)
```
利用定义的签名验证参数
```
def fun(*args, **kwargs):
    sig.bind(*args, **kwargs)
```
如果和签名不一致，会抛出异常。

实现一个装饰器用于检查类型
```
from functools import wraps


def sig_check(sig):
    def make_wrapper(fun):
        @wraps(fun)
        def _wrapper(*args, **kwargs):
            sig.bind(*args, **kwargs)
            return fun(*args, **kwargs)
        return _wrapper
    return make_wrapper
```

利用之前实例化的签名进行测试
```
@sig_check(sig)
def handler(*args, **kwargs):
    print(args, kwargs)


if __name__ == '__main__':
    handler(1, pk=2)
```
