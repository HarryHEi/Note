[文档](https://docs.python.org/3/library/functions.html)

# `property`装饰器
设置属性特性。

```
>>> class Battery(object):
...     def __init__(self):
...         self._v  = None
...     @property
...     def v(self):
...         return self._v        
...     
>>> b=Battery()
>>> b.v=10
Traceback (most recent call last):
  File "<input>", line 1, in <module>
AttributeError: can't set attribute

>>> class Battery(object):
...     def __init__(self):
...         self._v  = None
...     @property
...     def v(self):
...         return self._v
...     @v.setter
...     def v(self, value):
...         self._v = value
...         
>>> b=Battery()
>>> b.v = 10
>>> b.v
10
>>> del b.v
Traceback (most recent call last):
  File "<input>", line 1, in <module>
AttributeError: can't delete attribute

>>> class Battery(object):
...     def __init__(self):
...         self._v  = None
...     @property
...     def v(self):
...         return self._v
...     @v.setter
...     def v(self, value):
...         self._v = value
...     @v.deleter    
...     def v(self):
...         del self._v
...         
>>> b=Battery()
>>> b.v=10
>>> b.v
10
>>> del b.v
```

# `isinstance(object, classinfo)`

如果`object`是`classinfo`类或子类的实例，或者`object`是`classinfo`类型或子类型的对象，返回True。

# `issubclass(class, classinfo)`

如果`class` 是`classinfo`的子类，或者自己，返回True，`classinfo` 如果是元组，检查元组里面的每个元素，有一个满足则返回True。