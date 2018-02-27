---
title: Abstract Base Classes
date: 2018-02-27 15:05:03
tags: [python]
---


# `class abc.ABC`

一个含有`ABCMeta`作为metaclass的帮助类，使用这个类方便创建抽象基类。
```
class Empty(ABC):
	pass
```

相当于
```
class Empty(metaclass=ABCMeta):
	pass
```

# `class abc.ABCMeta`

用于定义抽象基类的metaclass

## `register(subclass)`

把一个类注册为抽象基类的子类
```
class Empty(ABC):
	pass

class Sub(object):
	pass

Empty.register(Sub)
```

这里Sub类就成为了Empty抽象基类的子类。

## `__subclasshook__(subclass)`

注意，必须被定义为类方法(class method)

如果返回True，那么该抽象基类的子类也是抽象基类，如果返回False，那么继承该抽象基类的类不是其子类(issubclass()返回False)。如果返回NotImplemented，子类检查使用通常机制。

# `@abc.abstractmentod`

一个抽象方法的装饰器。

只有metaclass是ABCMeta或继承ABCMeta的类可以使用这个装饰器。只有所有抽象基类都override之后才能实例化。只对常规继承的子类有用，使用register注册的方法没有影响。

有一个基类
```
class Base(metaclass=ABCMeta):
    @abstractmethod
    def some_fun(self):
        pass

    @classmethod
    def __subclasshook__(cls, subclass):
        return True
```

没有重写方法。
```
class Drived(Base):
    @classmethod
    def __subclasshook__(cls, subclass):
        return True


d = Drived()  # 错误，没有override所有抽象方法。
```

重写方法。
```
class Drived(Base):
    def some_fun(self):
        pass

    @classmethod
    def __subclasshook__(cls, subclass):
        return True


d = Drived()  # 没有问题，override所有abstractmethod。
```

使用register的没有影响。
```
class Drived(object):
    @classmethod
    def __subclasshook__(cls, subclass):
        return True


Base.register(Drived)

d = Drived()  # 对于register的类没有影响。
```
