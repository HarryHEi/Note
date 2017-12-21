[文档](https://docs.python.org/3/library/functions.html)

`property`装饰器
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