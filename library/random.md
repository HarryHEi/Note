[文档](https://docs.python.org/3/library/random.html)

本模块提供伪随机数生成器。

大部分功能都是以`random()`函数为基础的，生成[0.0, 1.0)的一个随机数。

# 函数
## `random.seed(a=None, version=2)`

如果`a`为None，使用系统时间作为seed；
如果`a`为一个整数，会被直接作为seed使用；
`version`默认为2，`str`、`bytes`或者`bytearray`对象会被转化成int使用。 
`version`为1是为旧版本提供，提供较小范围。

## `random.getrandbits(k)`

返回k个随机比特构成的整数，

# 用作整型的函数
## `random.randrange(stop)`
## `random.randrange(start, stop[, step])`

从`range(start, stop, step)`返回一个随机的数，相当于`choice(range(start, stop, step))`，但是不会真的构建一个`range`对象。

## `random.randint(a, b)`

返回 `a <= N <= b`的一个随机整数，是`randrange(a, b+1)`的语法糖。

# 用作序列的函数
## `random.choice(seq)`

返回序列中的一个随机元素，如果序列是空的，抛出`IndexError`。

## `random.choices(population, weights=None, *, cum_weights=None, k=1`

返回从列表中选取`k`个可重复的元素组成的列表。

如果指定`weights`，将更具列表的值设置每个元素的概率比重。

`cum_weigths`是累计概率比重，两个不能同时指定。

如果`weights`设为`[10, 20, 5, 10]`，相当于`cum_weights`设为`[10, 30, 35, 45]`

## `random.shuffle(x[, random])`

将序列`x`的位置重洗。

参数`random`是一个没有参数的函数参数，返回一个`[0.0, 1.0)`的整数，默认使用`random()`

如果需要不改变当前数组，返回一个新数组，使用`sample(x, k=len(x))`

## `random.sample(population, k)`

返回`k`个不重复元素组成的列表，不改变原序列。

如果序列含有重复的元素，每个重复的元素都只出现一次。

# 真值分布
## `random.random()`

返回下一个[0.0 1.0)随机数。

## `random.uniform(a, b)`

均匀分布，返回`a`和`b`之间的一个随机浮点数，`a <= N <= b`或者 `b <= N <= a`， 相当于`a + (b - a) * random()`

## `random.triangular(low, height, mode)`

三角分布，返回一个随机数`low <= N <= height`，并且带有特殊的`mode`，默认为0和1之间的中值。

## `random.betavariate(alpha, beta)`

β分布, 考虑参数`alpha > 0`以及`beta > 0`，返回0到1之间的随机值。

...
