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

# `all(iterable)`

如果可迭代参数的所有元素都为真，或者可迭代参数没有元素，返回True。

# `any(iterable)`

如果可迭代参数有一个参数为真，返回True，如果可迭代参数没有元素，返回False。

# `repr(object)`

返回一个对象的可打印字符串，大多数类型返回的值传递给`eval()`方法会获得该对象。
通常包含对象的名称和地址。对象可以通过`__repr__()`方法决定返回值。

```
>>> l = [1,2,3] 
>>> repr(l)
'[1, 2, 3]'

>>> q=queue.Queue()
>>> repr(q)
'<queue.Queue object at 0x00000235EB60B9E8>'
```

# `ascii(object)`

类似`repr()`，用`\x \u \U`替代其中的非ascii字符。

```
>>> class Test(object):
...     def __repr__(self):
...         return 'test.测试ascii()'
...     
>>> t=Test()
>>> repr(t)
'test.测试ascii()'
>>> ascii(t)
'test.\\u6d4b\\u8bd5ascii()'
```

# `format(value[, format_spec])`

根据`format_spec`构建一个值，(见`string.format`)。

`format_spec`默认情况下是个空串。

`format(value, format_spec)`被翻译为`type(value).__format__(value, format_spec)`

```
>>> format(123, 'x')
'7b'
>>> int.__format__(123, 'x')
'7b'
```

# `bin(x)`

转换为带有`0b`标识的二进制字符串，相当于`format(x, '#b')`

```
>>> bin(123)
'0b1111011'
>>> format(123, '#b')
'0b1111011'
```

# `class bool([x])`

返回一个真值，`x`用作标准的真值检测，如果`x`为false或者省略，返回False，否则返回True。
`bool`类是`int`类的子类，不能再被继承，只能被实例化为`False`和`True`。


# `class bytearray([source[, encoding[, errors]]])`

返回一个新的bytes数组，`bytearray`类是一个0~255整数组成的可变序列类型。
拥有大多数可变序列类型的方法，拥有`bytes`类型的大多数方法。

`source`参数用作初始化数组。
+ 如果是一个str，必须同时指定`encoding`参数，然后会使用`str.encode`
+ 如果是一个int，会初始化为对应擦汗年高度的空bytes
+ 如果是个类似buffer的实例，会被用于初始化为bytes数组。
+ 如果是个可迭代对象，元素必须是0~255的整型，然后用于初始化bytes数组。

如果没有参数，将初始化为空数组。

# `class bytes([source[, encoding[, errors]]])`

返回一个新的`bytes`对象，`bytes`对象是一个0~255整型组成的不可变序列，参数和`bytearray`类似。

#  `callable(object)`

如果对象有`__call__()` 方法，返回True，否则返回False。

# `chr(i)`

返回对应unicode code整型对应的字符。

# `@classmethod`

一个装饰器，把一个方法转为类方法。

类方法把类作为第一个参数，类似普通方法把类实例作为第一个参数。

类方法既可以由类调用，又可以由实例调用。

# `@staticmethod`

一个装饰器，把一个方法转为静态方法。

静态方法没有隐含的第一个参数。

静态方法既可以由了类调用也可以由实例调用。

静态方法与Java和C++的静态方法类似。

# `exec(object[, global[, locals]])`

动态执行python代码。

`object`必须是一个字符串或者一个代码对象。
如果是一个字符串，被作为python语句解析并执行。
如果是一个代码对象，直接执行。

允许进行文件IO。

如果省略后面的参数，默认使用当前环境。
```
>>> name='test'
>>> s='print(name)'
>>> exec(s)
test
```
如果只指定了`global`参数，`global`是一个字典，将被用于`global`和`local`变量。
```
>>> name = 'test'
>>> s='print(name)'
>>> scope = {}
>>> exec(s, scope)
Traceback (most recent call last):
  File "<input>", line 1, in <module>
  File "<string>", line 1, in <module>
NameError: name 'name' is not defined
```
通过`global`参数传递环境
```
>>> del name
>>> s='name="test"'
>>> scope = {}
>>> exec(s, scope)
>>> s='print(name)'
>>> exec(s,scope)
test
```
如果也指定了`local`参数
```
>>> global_scope = {}
>>> local_scope = {}
>>> exec('number = 10', global_scope)
>>> exec('number = 1', global_scope, local_scope)
>>> exec('print(number)', global_scope)
10
>>> exec('print(number)', global_scope, local_scope)
1
```

返回值为None。
