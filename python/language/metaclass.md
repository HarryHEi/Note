---
title: metaclass
date: 2018-02-27 14:41:00
tags: [python]
---


[参考](https://stackoverflow.com/questions/100003/what-are-metaclasses-in-python)

在Python中class是对象，是metaclass的实例。metaclass用于创建class。

使用type关键字可以查看一个对象的类型，也可以创建一个class对象。

创建一个空类
```
class Empty(object):
    pass
```

同样可以使用type创建一个空类
```
Empty = type('Empty', (), {})
```

type第一个参数是类名称，第二个参数是父类组成的元组，第三个参数是参数字典。

比如添加父类以及属性或者方法
```
def fun():
    pass

class Base():
    pass

Sub = type('Sub', (Base,), {'a': 1, 'fun': fun})
print(dir(d))
```

相当于
```
class Sub(Base):
    a = 1

    def fun():
        pass
```

这里的type就是一个metaclass。

当有个类
```
class Empty(Base):
    pass
```

创建一个class的主要步骤如下：

+ 首先检查Empty中是否有`__metaclass__`(python2)
+ 如果有，使用这个metaclass创建一个名为Empty的class对象。
+ 如果找不到`__metaclass__`，在模块级别查找，如果依然找不到，在Base中查找。

注意的是，`__metaclass__`不会被继承。

metaclass不一定是一个类，也可以是一个函数。

该函数通过调用type()方法创建一个类，并返回。
```
def UpperAttrMetaclass(cls_name, parents, attr):
    uppercase_attr = {}
    for name, value in attr.items():
        if not name.startswith('__'):
            uppercase_attr[name.upper()] = value
        else:
            uppercase_attr[name] = value

    return type(cls_name, parents, uppercase_attr)
```

一个把属性名称改为大写的metaclass
```
class UpperAttrMetaclass(type):
    def __new__(cls, cls_name, parents, attr):
        uppercase_attr = {}
        for name, value in attr.items():
            if not name.startwith('__'):
                uppercase_attr[name.upper()] = value
            else:
                uppercase_attr[name] = value

        return super(UpperAttrMetaclass, cls).__new__(cls, cls_name, parents, attr)
```

设置metaclass
```
class Sub(metaclass=UpperAttrMetaclass):
    name = 'test'

    def foo(self):
        pass


sub = Sub()
print(sub.NAME)  # 正常
print(sub.FOO())  # 正常
```
