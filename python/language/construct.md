---
title: 二进制数据处理
date: 2017-04-01 14:33:10
tags: [python]
---


`costruct`是python的二进制数据解析器和构造器

基本类型：

整数

Int{8,16,24,32,64}{u,s}{l,b,n}  对应选择位数、符号、字节序

Byte 等同于 Int8ub

浮点数

Float{32,64}{b,l,n}

Single、Double等同于Float{32,64}b

变长整型

VarInt

字段

Bytes

---

用Struct建立一个format

```
c1 = Struct(
    'a' / Byte,
    'b' / Int16ul,
    'c' / Single,
    'd' / Const(b'hello'),
)
```

传入一个字符串，解析成Container，Container就相当于一个普通的字典，并且具有具有自己的输出格式

```
p1 = c1.parse(b'\x07\x00\x01\x00\x00\x00\x01hello')
print p1
```

输出

```
Container: 
    a = 7
    b = 256
    c = 1.40129846432e-45
    d = hello
```

---

建立10个单字节的格式

```
c2 = Byte[10]
```

build函数的参数是个Container对象或者类似的鸭子类型（比如一个普通的类）

构建一个格式化的字符串，使用ord()获得对应的ASCII值输出查看效果

```
b2 = c2.build([1, 2, 3, 4, 5, 6, 7, 8, 9, 0])
print [ord(b) for b in b2]
```

输出

```
[1, 2, 3, 4, 5, 6, 7, 8, 9, 0]
```

解析刚才构建的字符串

```
p2 = c2.parse(b2)
print p2
```

输出

```
ListContainer: 
    1
    2
    3
    4
    5
    6
    7
    8
    9
    0
```

---

可以在Struct内放入Struct

```
c3 = Struct(
    'a' / Byte,
    'b' / Struct(
        'c' / Byte,
        'd' / Int16ul,
    )
)
```

传入Container对象构建

```
b3 = c3.build(Container(a=1, b=Container(c=2, d=3)))
print [ord(b) for b in b3]
```

输出

```
[1, 2, 3, 0]
```

解析之前构建的对象

```
p3 = c3.parse(b3)
print p3
```

输出
```
Container: 
    a = 1
    b = Container: 
        c = 2
        d = 3
```

---

可以通过>>构建格式

```
c4 = Int16ul >> Byte >> Bytes(5)
```

解析

```
p4 = c4.parse(b'\x01\x02\x03hello')
print p4
```

输出

```
ListContainer: 
    513
    3
    hello
```

Embedded用于嵌入格式
Switch类似于其他语言的switch语句，以一个lambda语句作为参数，lambda有个ctx上下文参数

```
from construct import *

header = Struct(
    'head' / Const(b'hello'),
)

frame = Struct(
    Embedded(header),
    'a' / Int8ul,
    'b' / Switch(lambda ctx: ctx['a'],
                 {
                     0x01: Int8ul,
                     0x02: Int16ul,
                     0x03: Int32ul,
                     0x04: VarInt,
                 })
)

b = frame.build(Container(head='hello', a=4, b=1231231312))
print [ord(c) for c in b]
```

输出

```
[104, 101, 108, 108, 111, 4, 208, 178, 140, 203, 4]
```

使用`this`关键字

```
frame = Struct(
    'length' / Int8ul,
    'payload' / Bytes(this.length)
)

b = frame.build(Container(length=2, payload=b'hi'))
print [ord(c) for c in b]
```

输出

```
[2, 104, 105]
```

# 参考

[文档](http://construct.readthedocs.io/en/latest/)

以上是2.8版本的，2.5版本略有不同[参考](http://www.thinksaas.cn/topics/0/435/435656.html)

---

在网络编程中也常使用struct模块用于数据的打包和解包

```
struct.pack('3B',1,2,3)
'\x01\x02\x03'
```

打包和解包

```
import struct

s = struct.pack('!3BI', 1, 2, 3, 4)
t = struct.unpack('!3BI', s)
print [ord(c) for c in s]
print t
```

输出

```
[1, 2, 3, 0, 0, 0, 4]
(1, 2, 3, 4)
```

在前面使用不同符号表示字节不同的位排序

![](http://7xrooc.com1.z0.glb.clouddn.com/QQ%E6%88%AA%E5%9B%BE20170406102515.png)

如果第一个字符不是当中的任何一个，自动添加一个`@`，启动本机序本机序是大端或者小端，取决于本机系统

`@`尺寸取决本机系统，`=`尺寸为标准大小

`<`小端 `>`大端 `!`网络序

各类型

![](http://7xrooc.com1.z0.glb.clouddn.com/QQ%E6%88%AA%E5%9B%BE20170406101144.png)

[文档](https://docs.python.org/2/library/struct.html)
[参考](http://www.cnblogs.com/coser/archive/2011/12/17/2291160.html)

---

socket提供函数用于ip地址字符串与整型之间的转换

```
>>> socket.inet_aton("192.168.1.1")
'\xc0\xa8\x01\x01'
>>> socket.inet_ntoa('\x01\x02\x03\x04')
'1.2.3.4'
```