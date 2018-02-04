---
title: numpy
date: 2016-04-01 20:00:58
tags: [python]
---

# numpy简介
numpy是一个多维数组对象（ndarray）
numpy数组的维度称为秩（rank），每个线性数组称为轴（axes）

+ ndarray.ndim:数组的维度秩（rank），即表示轴的个数
+ ndarray.shape:返回一个元祖，如果是二位数组，则表示“行数”和“列数”如果是如果是一个三通道图片，则表示“高”“宽”和“通道数”
+ ndarray.size:返回数组的元素的总个数，如果是三通道图片，值为`“高”*“宽”*“通道数3”`
+ ndarray.dtype:返回数组元素的数据类型对象，常见的比如uint8，float64
+ ndarray.itemsize:返回数组中每个元素大小，单位为字节，比如uint8是1字节，float64是8字节
+ ndarray.data:返回数组元素存储的缓冲区

# 创建一个numpy数组

## array()

使用array创建一个三维数组，注意参数要用括号括起来
```
>>> import numpy as np
>>> a = np.array([[[255,255,255]]])
>>> a.shape
(1, 1, 3)
>>> a[0][0]
array([255, 255, 255])
```
创建一个二位数组
```
>>> import numpy as np
>>> a = np.array([[1,2,3],[4,5,6]])
>>> a
array([[1, 2, 3],
       [4, 5, 6]])
>>> a.shape
(2, 3)

```
创建数组时可以显式指定元祖的数值类型
```
>>> a = np.array([[[255,255,255]]],dtype='uint8')
>>> a.dtype
dtype('uint8')
>>> a.itemsize
1
>>> a = np.array([[[255,255,255]]],dtype='float64')
>>> a.dtype
dtype('float64')
>>> a.itemsize
8
```

## arange()

返回一个数列形式数组
第一个参数表示开始的值，第二个参数表示截止的值(不包括)，第三个参数表示步长
```
>>> np.arange(1,10,2)
array([1, 3, 5, 7, 9])
```
第一个和第三个参数是可选的
```
>>> np.arange(7)
array([0, 1, 2, 3, 4, 5, 6])
>>> 
```
同样，也可以指定类型
```
>>> np.arange(7,dtype="uint8")
array([0, 1, 2, 3, 4, 5, 6], dtype=uint8)
```

# 自定义结构数组
```
>>> student = np.dtype({"names":["name", "age", "weight"], "formats":["S32", "i", "f"]}, align=True)
>>> student
dtype([('name', 'S32'), ('age', '<i4'), ('weight', '<f4')], align=True)
>>> a = np.array([("zhang", 32, 65.5), ("wang", 24, 55.2)], dtype = student)
>>> a
array([('zhang', 32, 65.5), ('wang', 24, 55.20000076293945)], 
      dtype={'names':['name','age','weight'], 'formats':['S32','<i4','<f4'], 'offsets':[0,32,36], 'itemsize':40, 'aligned':True})

```
## reshape
如果一个数组按照特定次序排列，可以使用reshape重定形
对一维数组重定形
```
>>> a = np.arange(9).reshape(3,3)
>>> a
array([[0, 1, 2],
       [3, 4, 5],
       [6, 7, 8]])
```
对二维数组重定形
```
>>> a = np.array([[1,2,3,4],[7,8,9,10]])
>>> a
array([[ 1,  2,  3,  4],
       [ 7,  8,  9, 10]])
>>> a.reshape(4,2)
array([[ 1,  2],
       [ 3,  4],
       [ 7,  8],
       [ 9, 10]])
```
## 水平组合
使用**hstack**函数
```
>>> a
array([[1, 2, 3, 4],
       [5, 6, 7, 8]])
>>> b
array([[ 9, 10, 11],
       [13, 14, 15]])
>>> np.hstack((a,b))
array([[ 1,  2,  3,  4,  9, 10, 11],
       [ 5,  6,  7,  8, 13, 14, 15]])
```
使用**concatenate**函数
```
>>> np.concatenate((a,b),axis=1)
array([[ 1,  2,  3,  4,  9, 10, 11],
       [ 5,  6,  7,  8, 13, 14, 15]])
```
## 垂直组合
使用**vstack**函数
```
>>> np.vstack((a,b))
array([[ 1,  2,  3,  4],
       [ 5,  6,  7,  8],
       [ 9, 10, 11, 12],
       [13, 14, 15, 16]])
```
使用**concatenate**函数
```
>>> np.concatenate((a,b),axis=0)
array([[ 1,  2,  3,  4],
       [ 5,  6,  7,  8],
       [ 9, 10, 11, 12],
       [13, 14, 15, 16]])
```
## 深度组合
使用**dstack**
```
>>> a
array([[1, 2, 3, 4],
       [5, 6, 7, 8]])
>>> b
array([[ 9, 10, 11, 12],
       [13, 14, 15, 16]])
>>> np.dstack((a,b))
array([[[ 1,  9],
        [ 2, 10],
        [ 3, 11],
        [ 4, 12]],
       [[ 5, 13],
        [ 6, 14],
        [ 7, 15],
        [ 8, 16]]])
```
## 行组合
```
>>> a
array([[1, 2],
       [3, 4]])
>>> row_stack((a,a))
>>> np.row_stack((a,a))
array([[1, 2],
       [3, 4],
       [1, 2],
       [3, 4]])
```
## 列组合
```
>>> np.column_stack((a,a))
array([[1, 2, 1, 2],
       [3, 4, 3, 4]])
```

## 水平分割
```
>>> a=np.arange(9).reshape((3,3))
>>> a
array([[0, 1, 2],
       [3, 4, 5],
       [6, 7, 8]])
>>> np.hsplit(a,3)
[array([[0],
       [3],
       [6]]), array([[1],
       [4],
       [7]]), array([[2],
       [5],
       [8]])]
```
```
>>> np.split(a,3,axis=1)
[array([[0],
       [3],
       [6]]), array([[1],
       [4],
       [7]]), array([[2],
       [5],
       [8]])]
```
## 垂直分割
```
>>> a
array([[0, 1, 2],
       [3, 4, 5],
       [6, 7, 8]])
>>> np.vsplit(a,3)
[array([[0, 1, 2]]), array([[3, 4, 5]]), array([[6, 7, 8]])]
```
```
>>> np.split(a,3,axis=0)
[array([[0, 1, 2]]), array([[3, 4, 5]]), array([[6, 7, 8]])]
```
## 深度分割
```
>>> a = np.arange(8).reshape(2,2,2)
>>> a
array([[[0, 1],
        [2, 3]],

       [[4, 5],
        [6, 7]]])
>>> np.dsplit(a,2)
[array([[[0],
        [2]],

       [[4],
        [6]]]), array([[[1],
        [3]],

       [[5],
        [7]]])]
```
# 复制和镜像
## 简单的赋值
```
>>> a = np.array([1,2,3,4])
>>> b = a
>>> b is a
True
>>> a.shape=2,2 #a形状改变
>>> a
array([[1, 2],
       [3, 4]])
>>> b
array([[1, 2],	#b形状也改变了
       [3, 4]])
```
## 视图和浅复制
```
>>> c = a.view()
>>> c is a
False
>>> c.base is a
True
>>> a
array([[1, 2],
       [3, 4]])
>>> c.shape=4, #c形状改变
>>> c
array([1, 2, 3, 4])
>>> a
array([[1, 2], #a形状没有改变
       [3, 4]])
>>> c[0]=100 #c数值改变
>>> a
array([[100,   2], #a数值也改变了
       [  3,   4]])
```
```
>>> a=np.arange(10)
>>> a
array([0, 1, 2, 3, 4, 5, 6, 7, 8, 9])
>>> b=a[:5]
>>> b
array([0, 1, 2, 3, 4])
>>> a[:]=10
>>> a
array([10, 10, 10, 10, 10, 10, 10, 10, 10, 10])
>>> b
array([10, 10, 10, 10, 10])
```
## 深复制
```
>>> b = a.copy()
>>> a
array([10, 10, 10, 10, 10, 10, 10, 10, 10, 10])
>>> b
array([10, 10, 10, 10, 10, 10, 10, 10, 10, 10])
>>> a[:]=9
>>> a
array([9, 9, 9, 9, 9, 9, 9, 9, 9, 9])
>>> b
array([10, 10, 10, 10, 10, 10, 10, 10, 10, 10])
```
# json

## 简介
JSON(JavaScript Object Notation) 是一种轻量级的数据交换格式。易于人阅读和编写。同时也易于机器解析和生成。
ndarray不能被json识别，先使用ndarray.tolist()转化成list，然后在转化成json
```
import cv2
import numpy as np
import json

#创建一个numpy.ndarray
a = np.array([[[255,255,255]]])
print "1:",type(a)

#ndarray转化成list
s = a.tolist()
print "2:",type(s)

#json编码
j = json.dumps(s)
print "3:",type(j)

#json解码
l = json.loads(j)
print "4:",type(l)

#list转化为numpy.ndarray
c = np.array(l)
print "5:",type(c)
```
输出结果
```
1: <type 'numpy.ndarray'>
2: <type 'list'>
3: <type 'str'>
4: <type 'list'>
5: <type 'numpy.ndarray'>
```
## Json 自定义
```
import json

class Student():
    def __init__(self, name, age, score):
        self.name = name
        self.age = age
        self.score = score

def Student2Dic(std):
    return {
        "name":std.name,
        "age":std.age,
        "score":std.score
    }

def Dic2Student(d):
    return Student(d['name'], d['age'], d['score'])

s = Student("harry", 22, 90)
j = json.dumps(s, default=Student2Dic)
print type(j),j
d = json.loads(j, object_hook=Dic2Student)
print type(d),d
```
输出
```
<type 'str'> {"age": 22, "score": 90, "name": "harry"}
<type 'instance'> <__main__.Student instance at 0xb70cd3cc>
```

# 数值类型

| 类型 | 说明 |
|-------|--------|
| bool | 用一个字节存储的布尔类型（True或False） |
| inti | 由所在平台决定其大小的整数（一般为int32或int64） |
| int8 | 一个字节大小，-128 至 127 |
| int16 | 整数，-32768 至 32767 |
| int32 | 整数，-2 **31** 至 2 **32 -1** |
| int64 | 整数，-2 **63** 至 2 **63 - 1** |
| uint8 | 无符号整数，0 至 255 |
| uint16 | 无符号整数，0 至 65535 |
| uint32 | 无符号整数，0 至 2 ** 32 - 1 |
| uint64 | 无符号整数，0 至 2 ** 64 - 1 |
| float16 | 半精度浮点数：16位，正负号1位，指数5位，精度10位 |
| float32 | 单精度浮点数：32位，正负号1位，指数8位，精度23位 |
| float64或float | 双精度浮点数：64位，正负号1位，指数11位，精度52位 |
| complex64 | 复数，分别用两个32位浮点数表示实部和虚部 |
| complex128或complex | 复数，分别用两个64位浮点数表示实部和虚部 |

# 参考
<http://blog.csdn.net/sunny2038>
<http://www.cnblogs.com/coser/archive/2011/12/14/2287739.html>
