`collections` 容器类型模块

[官方文档](https://docs.python.org/3/library/collections.html)

# `deque`对象

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
