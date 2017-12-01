# 参考文档
[Built-in Types](https://docs.python.org/3.6/library/stdtypes.html)

# 真值检测
所有对象都可以通过`if`或者`while`进行真值检测，在默认情况下，反回`True`。
可以通过定义`__bool__()`方法决定其真值。

# 比较
在Python中有8个比较符(comparison operation)：

## `<`、`<=`、`>`、`>=`符号
这些符号不能比较两个不同类型，否则会`TypeError`异常。
方法：

+ `x.__lt__(y)`相当于`x<y`
+ `x.__le__(y)`相当于`x<=y`
+ `x.__gt__(y)`相当于`x>y`
+ `x.__ge__(y)`相当于`c>=y`

PS：当定义了相应比较方法后，才可以进行排序操作。

## `==`、`!=`符号
默认情况下，`id`不通的两个同一对象的实例比较结果为不等。
可以通过定义`__eq__()`方法自定义对象的比较方式。

## `is`、`is not`
`is`和`is not`符号操作不能被自定义，但是可以用于两个不同的对象，且不会抛出异常。

## `in`、`not`
还有两个语法相似的符号`in`和`not in`，只能应用于序列类型(sequence types)

# 数值类型
比较明显的三种数值类型：整数类型(int)，浮点类型(float)，复数类型(complex)。
另外，布尔类型(bool)是特殊的整数类型。

整数类型具有无限精度。

浮点类型相当于 C 里面的`double`类型。具体精度细则可以通过`sys.float_info`获知。

复数类型具有实数部分(`real`)和虚数部分(`imag`)，虚数后添加一个`j`或者`J`符号。

Python完全支持混合计算。

Tips：

1. 负数使用整除时会被翻译为负无穷。
`(-1)//2=>-1`、`1//(-2)=>-1`、`(-1)//(-2)=>0`
2. 复数也可以使用`abs()`计算绝对值.
3. 浮点小数到整数的转换和 C 类似。
`round()`用于四舍五入，`math.trunc()`截断，`math.floor()`向下取整和`math.ceil()`向上取整。
4. float类型可以接收`nan`和`inf`(可以前置+或-)字符串表示`NaN`(Not a Number)非数值和正负无穷。
5. `0 ** 0 => 1`

## 整型位操作
只有整数类型可以进行位操作，负数的进行位操作时取补码值。

优先值比数值操作符低，比比较操作符高。

Tips：

1. 移位值位负数会抛出`ValueError`异常，例如`1 << -2`。
2. 没有溢出的话，左移`n`位相当于乘以`power(2, n)`。
3. 没有溢出的话，右移`n`位相当于除以`power(2, n)`。

## 整型的其他方法
### `int.bit_length()`
返回整型数值转换成2进制所需最少位数。
```
int(123).bit_length()  # 输出7，123相当于0b1111011。
```

### `int.to_bytes(length, byteorder, *, signed=False)`
返回表示整型数值的`bytes`字符串
```
>>> int(20).to_bytes(2, byteorder='big')
b'\x00\x14'
>>> int(20).to_bytes(2, byteorder='little')
b'\x14\x00'
```
如果所给位数`length`不够用来表达整数类型，会抛出`OverflowError`异常。

其中`byteorder`参数决定高位先行还是低位先行，或者使用系统相关`sys.byteorder`作为参数。

`signed`参数决定是否表示负数，如果`signed=False`时，尝试转换一个负数会抛出`OverflowError`异常。
```
int(-1).to_bytes(2, byteorder='little')
Traceback (most recent call last):
  File "<input>", line 1, in <module>
OverflowError: can't convert negative int to unsigned
int(-1).to_bytes(2, byteorder='little', signed=True)
b'\xff\xff'

```

### `classmethod int.from_bytes(bytes, byteorder, *, signed=False)`
将所给bytes字符串转换位整型。

## 浮点型的其他方法
### `float.as_integer_ratio()`
返回一对比例等于这个浮点数的整数。

如果尝试转换`NaN`，会抛出`ValueError`异常。

### `float.is_integer()`
如果浮点类型只限为整数值，返回`True`。

## 数值类型的哈希值
对于数值`x`和`y`，如果`x == y`，那么`hash(x) == hash(y)`。

# 迭代器类型
迭代器是基于容器的概念，为了使用户自定义的类支持迭代器，定义以下两种方法

## `container.__iter__()`
返回一个迭代器对象，返回的迭代器对象提供以下两个协议。如果容器提供各种不同类型的迭代器，可以使用额外的方式获取迭代器。

### `iterator.__iter__()`
返回迭代器本身，使容器和迭代器支持`for`和`in`操作。

### `iterator.__next__()`
返回容器的下个元素，如果没有更多的元素，将会触发一个`StopIteration`异常。
```
>>> lis = [1,2,3]
>>> iter = lis.__iter__()
>>> iter.__next__()
1
>>> iter.__next__()
2
>>> iter.__next__()
3
>>> iter.__next__()
Traceback (most recent call last):
  File "<input>", line 1, in <module>
StopIteration
```

# 序列类型
## 一般序列类型
有三个基本的序列类型：`list`、`tuple`、`range`。两个特殊的一般序列`str`、`bytes`。

Tips：

分清序列的元素是拷贝还是引用。
```
>>> lis = [[]]*3
>>> lis
[[], [], []]
>>> lis[0].append(1)
>>> lis
[[1], [1], [1]]
```
这里的`[[]]*3`并不等同于`[[]] + [[]] + [[]]`，而相当于：
```
>>> l=[]
>>> lis = [l,l,l]
```
因此以上方式是元素的引用，可以采用下面的方式创建不同数组组成的数组：
```
>>> lis = [[] for i in range(5)]
>>> lis
[[], [], [], [], []]
>>> lis[0].append(1)
>>> lis
[[1], [], [], [], []]
```

## 不可变序列类型
不可变序列通常执行的区别于可变序列操作是支持`hash()`。

这个支持使得不可变类型，比如`tuple`实例可以用作`dict`的键值，或者储存在`set`和`frozenset`中。
```
>>> d={}
>>> d['a']=1
>>> d[('b', 'c')]=2
>>> d
{'a': 1, ('b', 'c'): 2}
>>> d[['d', 'e']]=3
Traceback (most recent call last):
  File "<input>", line 1, in <module>
TypeError: unhashable type: 'list'
```
但是尝试获取包含不能获取哈希值的不可变类型序列也将抛出`TypeError`异常。
```
>>> hash((1,2,3))
-378539185
>>> hash((1,2,[3]))
Traceback (most recent call last):
  File "<input>", line 1, in <module>
TypeError: unhashable type: 'list'
```

## 可变序列类型
Tips：

1. 如果元素不存在，`remove`会抛出`ValueError`异常。
2. 为了保证效率，`reverse()`方法进行地址操作，因此不会返回新的数组。
3. 为了保证与不支持切片的序列类型的一致性，保留了`clear`和`copy`操作。

## Lists
列表是可变序列类型，一般用于保存多个同类型元素。

### `sort(self, key=None, reverse=False)`
将数组排序，需要元素之间支持`<`比较。

`key`参数指定一个函数提取用于比较的元素，每个元素只计算一次，默认直接使用元素值比较。比如`key=str.lower`，用于不区分大小写排序。

`reverse`如果设置为`True`，反向排序。

## Tuples
Tuples是一个不可变序列，一般用于保存各种不同类型数据。

## range()
`class range(stop)`
`class range(start, stop[,step])`

# 文本序列类型
Python中把文本数据处理为`str`对象或者字符串，字符串是unicode编码的不可变序列（字面值常量）。

三引号字符串一般跨越多行用连续的空格隔开。

字符串字面值是单个表达式，空格隔开的多个字符串会被处理为一个字符串字面值
```
>>> ("hello " "world") == "hello world"
True
```
为了向前兼容，允许添加`u`前缀，虽然没有什么影响，但是不能和`r`前缀组合。

`class str(object='')`
`class str(object=b'', encoding='utf-8', errors='strict')`

如果没有指定`encoding`和`errors`参数`str(object)`返回`object.__str__()`的值。
如果没有定义`__str__()`方法，会返回`repr(object)`，返回对象的基本描述。

如果指定了`encoding`或者`errors`，`object`必须是`bytes-like object`(bytes, bytearray, array.array)。如果`object`是`bytes`，相当于`bytes.decode(encoding, errors)`。

如果传入`bytes` 参数，没有指定`encoding`和`errors`参数，会进入第一种情况，返回对象的基本描述。

## 字符串方法
字符串支持所有一般序列的操作。

字符串支持两种风格构造格式，一种是用`str.format`，另外一种是 C `printf`风格。

### `str.capitalize()`
返回字符串的拷贝，第一个字母大写，其余小写。

### `str.casefold()`
返回字符串小写的拷贝，且支持其他语言。

### `str.center(width[, fillchar])`
返回固定长度字符串，该字符串放在中间，其余默认补空格，或者通过参数`fillchar`指定。
```
>>> 'hello'.center(20)
'       hello        '
```
### `str.count(sub[, start[, end]]`
返回`sub`指定的子串出现的次数。

### `str.encode(encoding='utf-8', errors='strict')`
返回字符串编码后的版本。`errors`参数指定编码错误后的错误处理方式，`strict`表示抛出`UnicodeError`，其他还有`ignore`，`replace`，`xmlcharrefreplace`，`backslashreplace`或者使用`codecs.register_error()`注册的处理方式。

### `str.endswith(suffix[, start[, end]])`
如果以`suffix`指定的字符串结尾，则返回true，否则返回false。`suffix`也可以是个元组，用于查找。

### `str.find(sub[, start[, end]])`
返回能够查找到`sub`指定子串的最小索引。

### `str.format(*args, **kwargs)`
构造一个字符串。
格式说明：
```
'{' [field_name] ["!" conversion] [":" format_spec] '}'
```
其中`field_name`结构为：`arg_name("." attribute_name | "[" element_index "]")`

`arg_name`是一个数字或者关键字，如果是一个数字，表示的是一个位置。如果是一个关键字，可以通过`format`字典参数指定关键字。

`.attribute`对参数进行点操作。

`!conversion`指定在构造之前进行过一个格式操作。
`!s`调用`str()`，`!r`调用`repr()`，`!a`调用`ascii()`

`format_spec`指定值的显示方式，格式入下：
```
[[fill] align] [sign] [#] [0] [width] [grouping_option] [.precision] [type]
```
`fill`指定填充字符，可以是任意字符。

`align`指定对其方式，其中`<`(左对齐)，`>`右对齐，`=`在`+`或`-`符号之后数字符号之前填充，`^`居中。

`sign`指定符号显示方式，其中`+`正数也会显示`+`符号，`-`只有负数会显示`-`符号，正数不显示，空格同`-`。

`#`会使得辅助符号显示，只`int`、`float`、`complex`、`decimal.Decimal`有效。对于整数类型，会添加比如`0b`、`0o`和`0c`符号。对于其他几个，会添加一个小数点，即使并没有小数部分。
```
>>> '{:#}'.format(1+2j)
'(1.+2.j)'
```
如果添加了`0`标识，并且通过`width`参数指定了长度，数值前的空缺位自动补0。
```
>>> '{:#010}'.format(123.321)
'000123.321'
```
`grouping_option`如果指定为`,`，整数位每千位会用逗号隔开。如果指定为`_`，则用下划线隔开。
```
>>> '{:,}'.format(12313123123.3421323)
'12,313,123,123.342133'
>>> '{:_}'.format(1233213213)
'1_233_213_213'
```
`precision`指定小数点后显示多少位。

`type`如下：

对于整数类型：

+ `b`，二进制格式；
+ `c`，整型转换成unicode字符；
+ `d`，十进制整数；
+ `o`，八进制；
+ `x`，十六进制小写；
+ `X`，十六进制大写；
+ `n`，和`d`一样。

对于浮点类型

+ `e`，指数格式；
+ `E`，同`e`，使用大写的E；
+ `f`，小数固定6位，不足补0；
+ `F`，同上，但是`nan` 转为`NAN`，`inf`转为`INF`；
+ `g`，固定显示数的数量，数量由`precise`指定。`'{a:.2g}'.format(a=123.32132) ==> '1.2e+02'`；
+ `G`，同上，使用大写的E；
+ `n`，同`g`；
+ `%`，乘以100，再相当于加上`f`，以百分号结尾。`'{a:%}'.format(a=0.676766) ==> '67.676600%'`

### `str.format_map(mapping)`
和`str.format(**mapping)`类似，直接传入一个字典参数。
```
>>> d=dict(a=1)
>>> '{a}'.format_map(d)
'1'
>>> '{a}'.format(d)
Traceback (most recent call last):
  File "<input>", line 1, in <module>
KeyError: 'a'
```
### `str.index(sub[, start[, end]])`
和`find()`类似，但是如果没有找到会抛出`ValueError`遗异常。

### `str.isalnum()`
全部都是字符或者数字，否则返回false。

### `str.isalpha()`
全是字母，否则返回false。

### `str.isdecimal()`
全是全角数字或者Unicode数字，否则返回false。

### `str.isdigit()`
全是Unicode数字、bytes数字、全角数字或者罗马数字，否则返回false。

### `str.isnumeric()`
全是Unicode数字、全角数字、罗马数字或者汉字数字。

### `str.isidentifier()`
如果是合法的标识符，返回true。
