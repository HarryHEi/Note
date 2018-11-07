---
title: collections
date: 2017-12-24 21:33:37
tags: [python]
---

[文档](https://docs.python.org/3/library/collections.html)

这个模块实现了专用的容器类型以代替Python的内建容器dict、list、set和tuple。

# deque

和`list`内置类型相似的容器，可以从两端添加弹出元素吗，线程安全。

## `class deque([iterable[, maxlen]])`

指定`iterable`时，从左到右取数据初始化，不指定则初始化为空。

指定`maxlen`时，限制长度，已满状态下添加新元素将从相反一端丢弃相应数量元素，不指定则不限制长度。

## 支持的方法

`append(x)`，从右端添加元素

`appendleft(x)`，从左端添加元素

`clear()`，删除所有元素

`copy()`，创建一个浅拷贝

`count(x)`，计算与x相等的元素数

`extend(iterable)`，右端添加元素

`extendleft`，左端添加元素

`index(x[, start[, stop]])`，返回第一个与x相等的元素索引，没有则抛出`ValueError`

`insert(i, x)`，在i位置插入x，如果i超过总长度，抛出`IndexError`

`pop()`，弹出右端元素，没有元素则抛出`IndexError`

`popleft()`，弹出左端元素，没有元素则抛出`IndexError` 

`remove(value)`，移除第一个出现的value，如果没有，则抛出`ValueError`

`reverse()`，反转元素，返回None

`rotate(n)`，右旋n个位置，负数则左旋。

`maxlen`，长度限制，或者None

## 示例

限制长度的`deque`可以提供类型Unix中`tail`的功能

```
>>> msg = 'a very very long string'
>>> def tail(s, n=5):
...     return deque(s, n)
... 
>>> tail(msg)
deque(['t', 'r', 'i', 'n', 'g'], maxlen=5)
```

删除`deque`中的某个位置元素，类似`del lis[n]`的功能

```
>>> def delete_nth(d, n):
...     d.rotate(-n)
...     d.popleft()
...     d.rotate(n)
>>> d
deque([1, 2, 3, 4, 5])
>>> delete_nth(d, 2)
>>> d
deque([1, 2, 4, 5])
```

# namedtuple

`collections.namedtuple(typename, field_names, *, verbose=False, module=None)`

用以构建只有少数属性但是没有方法的对象，比如数据库条目。例如创建一个纸牌对象
```
>>> import collections
>>> Card = collections.namedtuple('Card', ['rank', 'suit'])
```

namedtuple赋予元组中每个位置含义，常规元组可以使用的地方都能使用，并且具有通过名称索引的能力。

`typename`为返回的namedtuple的名称。

`field_names`是字符串序列，也可以是以空白或者逗号隔开的字符串，比如`'x y'`以及`'x, y'`。字段名称不得以下划线或者数字开始，不能是Python关键字。

如果`verbose`为True，类定义会在构建后打印出来，或者通过`_source`属性打印。

如果`rename`为True，无效的字段名会自动替换为位置名称。形如`_1`。

如果定义了`module`，`__module__`会被设置为该值。

`_fields`属性是包含这个类所有字段名称的元组。
```
>>> Card._fields
('rank', 'suit')
```

`_make()`方法接受一个可迭代对象来生成这个类的一个实例，作用和`namedtuple(*iterable)`一样。
```
>>> Card._make([1, 'cube'])
Card(rank=1, suit='cube')
>>> Card(*[1, 'cube'])
Card(rank=1, suit='cube')
>>> Card(1, 'cube')
Card(rank=1, suit='cube')
```

`_asdict()`把元组以collections.OrderedDict的形式返回，用于显示元组的信息。
```
>>> card = Card(1, 'cube')
>>> card._asdict()
OrderedDict([('rank', 1), ('suit', 'cube')])
```

# collection.defaultdict

defaultdict在找不到key时可以根据给出的方法创建默认值。

创建defaultdict时传入一个可调用对象，这个可调用对象会存入`default_factory`，，当`__getitem__()`方法知道不到key值时，会调用这个可调用对象并存下并返回值。

如果没有指定default_factory，当找不到key时会抛出KeyError异常。

注意default_factory只有在`__getitem__()`方法找不到key时调用，在`get()`方法中只会返回None。
