---
title: 内建函数
date: 2017-12-21 8:45:37
tags: [python]
---

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
如果也指定了`local`参数，则`global`和`local`分别作为全局和局部环境。
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

语法错误通过异常报告。

返回值为None。

# `eval(expression, globals=None, locals=None)`

参数和`exec()`类似，不同的是，返回值是表达式的值。

# `compile(source, filename, mode, flags=0, dont_inherit=False, optimize=-1)`

将源代码构造为代码或者抽象代码树对象。

代码对象可以由`exec()`和`eval()`函数执行。

源代码可以是普通字符串、byte字符串或AST(抽象代码树)。

`filename`参数指定代码文件，如果不从文件读，可以传递一个容易识别的值，比如空字符串。

`mode`参数指定编码的类型。`exec`顺序执行操作，`eval`单一表达式，`signal`处理单一交互。

如果指定为`exec`并且传递给`eval()`函数，将返回None。

# `getattr(object, name[, default])`

返回对象的属性，`name`是一个字符串。如果name参数是对象的某个属性，返回该值，如果属性不存在，且指定了`default`参数，返回该值，则否则抛出`AttributeError`。
`getattr(t, 'test')`等同于`t.test`

# `setattr(object, name, value)`

设置对象属性，`name`参数可以是对象的属性或者是一个新属性。
`setattr(t, 'test', 123)`等同于`t.test = 123`

# `delattr(object, name)`

删除对象属性，如果属性不存在，抛出`AttributeError`。
`delattr(t, 'test')`等同于`del t.test`

# `dir([object])`

不加参数的情况下，返回当前环境的所有名称。
有一个参数时，返回该对象的所有属性。

如果参数对象含有`__dir__()`方法，该方法会被调用，比且必须返回一个属性组成的数组。

默认`dir()`这对不同类型的参数有不同的行为。
如果`object`是一个模块对象，将列出模块属性名称。
如果`object`是类型或者类对象，将列出他们的以及基类的属性名称。
其他情况，列出对象属性名称、类属性名称、基类属性名称。

结果按照字符顺序列出。

# `divmod(a, b)`

对于整数，结果相当于`(a // b, a % b)`。
对于浮点数， 结果相当于`(math.floor(a / b), a % b)`，`0 <= abs(a % b) < abs(b)`

# `enumerate(iterable, start=0)`

返回一个枚举对象，`iterable`必须是一个序列、迭代器或者其他支持迭代操作的对象。

枚举对象的`__next__()`方法将返回一个元组，包括一个从0开始的计数和`iterable`迭代的值。

# `filter(function, iterable)`

构造一个迭代器，包含`iterable`中所有使得`function`返回True的元素。

`iterable`可以是序列、支持迭代的容器或者迭代器。

如果`function`是None，`iterable`中所有值为False的元素将被移除。

```
>>> lis = [0, 1, 2, [], '', ',']
>>> [v for v in filter(None, lis)]
[1, 2, ',']
```
