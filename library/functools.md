functools模块用于高阶函数（表现为返回其他函数的函数）。

`function.redece(function, iterable[, initializer])`
把接收两个参数的函数function用在iterable的每个元素上，每次的结果在和下一个元素累计调用。

```
>>> add = lambda a, b: a + b
>>> reduce(add, [1,2,3,4,5,6])
21
```
