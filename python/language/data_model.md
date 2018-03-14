---
title: 数据模式
date: 2018-1-8 20:40:00
tags: [python]
---

[文档](https://docs.python.org/3/reference/datamodel.html)

# 特殊方法名

通过定义特殊名称的方法，类可以使用特定语法实现特定操作。这是Python的运算符重载，允许类定义自己的运算符行为。可以设置一个特殊方法为None标识相应操作不可用。

## 基础定制方法

`object.__new__(cls[, ...])`

一个静态方法，用于创建一个类的实例，第一个参数是这个实例的类。剩下的参数用于传递给对象的构造表达式，然后由类调用。返回值应该是一个新的实例，一般返回`cls`的实例。

典型的实现是传入适当的参数，通过调用父类的`__new__()`方法创建实例，形如`super().__new__(cls[, ...])`，然后在返回之前对实例进行修改。

如果`__new__()`返回`cls`的实例，然后实例的`__init__()`方法会被调用，形如`__init__(self[, ...])`其中`self`是新创建的实例。

如果`__new__()`不返回`cls`的实例，那么实例的`__init__()`方法不会被调用。

`__new__()`主要用于允许不可变类型（比如int，str或者tuple）的子类创建自定义实例。通常在自定义元类中为了创建自定义元类被重写。

可以用于实现单例模式
```
class SingleClass(object):
    def __new__(cls, *args, **kwargs):
        if not hasattr(cls, '_ins'):
            cls._ins = super(SingleClass, cls).__new__(cls, *args, **kwargs)
        return cls._ins

    def __init__(self):
        print('do init')
```
测试一下
```
>>> s=SingleClass()
do init
>>> c=SingleClass()
do init
>>> s
<main.SingleClass object at 0x000001F10EDFE390>
>>> c
<main.SingleClass object at 0x000001F10EDFE390>
```
虽然实现了单例模式，但是注意的是，每次创建一个实例都会调用一次`__init__()`方法。

`object.__init__(self[, ...])`

当实例被创建后(也就是`__new__()`方法调用之后)，返回给调用者之前调用。参数是传递给类构造器的参数。如果基类含有一个`__init__()`方法，派生类的`__init__()`方法必须显式调用父类的`__init__()`方法以确保基类部分正确初始化。比如`super(Derived, self).__init__([args...])`。

`object.__del__(self)`

当实例即将被销毁时调用，如果基类有`__del__()`方法，派生类的`__del__()`方法必须显式调用父类的`__del__()`方法以确保销毁实例的父类部分。

可以同过在`__del__()`方法中再创建一个对象的引用来"复活"对象（不推荐）。

并不保证在解释器退出时`__del__()`方法还存在。

`del x`并不直接调用`x.__del__()`，只是将引用计数减1，当引用计数为0时才调用。

`object.__repr(self)`

由`repr()`方法以及`'%r'%object`调用，用于返回一个对象的可打印字符串，返回值传递给`eval()`可以获得该对象。

`object.__str__(self)`

由`str(object)`、`format()`和`print()`调用，将对象进行字符串表示，返回值必须是一个字符串对象。和`__repr__()`不同的是，`__str__()`不一定返回有效的Python表达式。当没有定义`__str__()`时，解释器会用`__repr__()`代替。

`object.__bytes__(self)`

由bytes类调用，计算对象的bytes字符串表示，应该返回一个bytes对象。比如将一个数值加上头和尾返回bytes字符串。

```
class Test(object):
    def __init__(self, val):
        self.val = val

    def __bytes__(self):
        return bytes([0x00, int(self.val), 0x00])

>>> t=Test(1)
>>> bytes(t)
b'\x00\x01\x00'
```

`object.__format__(self, format_spec)`

由`format()`调用，`format(15, '02x')`相当于`int.__format__(15, '02x')`，返回`0f`。

`object.__lt__(self, other)`
`object.__le__(self, other)`
`object.__eq__(self, other)`
`object.__ne__(self, other)`
`object.__gt__(self, other)`
`object.__ge__(self, other)`

各种比较操作。

`object.__hash__()`

由`hash()`调用，如果x==y，那么hash(x) == hash(y)。

## 属性接口

属性接口用于获取类实例的属性。

`object.__getattr__(self, name)`

当通过通常的方式没有找到属性时(即不是实例的属性也不是类属性)调用。

`object.__getattribute__(self, name)`

当需要获取参数时无条件执行。
如果同时定义了`__getattr__()`和`__getattribute__()`，除非`__getattribute__()`显式指定或者抛出`AttributeError`，否则`__getattr__()`将不会被调用。
