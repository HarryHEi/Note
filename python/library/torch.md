---
title: torch
date: 2018-2-4 21:41:37
tags: [python]
---

[文档](http://pytorch.org/tutorials/beginner/blitz/tensor_tutorial.html)

# Tensors

Tensors和NumPy的ndarrays(比如由`numpy.array()`方法创建的实例)类似，并且可以在GPU上加速计算。

[所有支持的操作](http://pytorch.org/docs/master/torch.html)

# autograd

PyTorch神经网络最核心的是autograd包，提供支持Tensors所有操作方式的自动区分，是一个运行时定义的框架。

## Variable

`autograd.Variable`是这个包的核心，包装了一个Tensor，支持Tensor的所有操作。一旦完成计算，可以调用`.backward()`自动计算梯度。

可以通过`.data`属性获取原始数据，梯度可以通过`.grad`属性获取。

还有一个非常重要的类`Function`，和`Variable`连接并建立非环状图表，编写完整的计算历史。每个Variable有一个`.grad_fn`属性保存创建这个Variable的Function的引用，除非是由用户创建的Variables，它们的`grad_fn`是None。

如果想要计算衍生物，可以调用`Variable.backward()`，如果`Variable`是标量，比如一元数据，`backward()`就不需要指定参数，否则，必须指定`grad_output`参数匹配Tensor形状。

创建一个Variable
```
In[31]: x=Variable(torch.ones(2,2), requires_grad=True)
In[32]: x
Out[32]: 
Variable containing:
 1  1
 1  1
[torch.FloatTensor of size 2x2]
```

进行一些操作
```
In[33]: y = x * 2
In[34]: y
Out[34]: 
Variable containing:
 2  2
 2  2
[torch.FloatTensor of size 2x2]
```

这里y是由计算的结果创建的，所有它具有`grad_fn`参数
```
y.grad_fn
Out[35]: <torch.autograd.function.MulConstantBackward at 0x1eb95684618>
```

## Gradients

做一些操作，这里out是计算z的平均值:
```
In[36]: z=y*y*3
In[37]: z
Out[37]: 
Variable containing:
 12  12
 12  12
[torch.FloatTensor of size 2x2]
In[38]: out=z.mean()
In[39]: out
Out[39]: 
Variable containing:
 12
[torch.FloatTensor of size 1]
```

回过头调用`out.backward()`也等同于`out.backward(torch.Tensor([1.0]))`

```
In[40]: out.backward()
In[41]: x.grad
Out[41]: 
Variable containing:
 6  6
 6  6
[torch.FloatTensor of size 2x2]
```

已知out等于z的平均值，z等于y的平方乘以3，y等于2乘以x，并且当x等于1时z等于12。

列出out和x的表达式: `out = 12x^2` ，求导得: `6`

与最后的结果相同。


范数:
```
In[42]: x=torch.randn(3)
In[43]: x=Variable(x, requires_grad=True)
In[44]: y=x*2
In[45]: while y.data.norm() < 1000:
   ...:     y = y * 2
   ...:     
In[46]: y
Out[46]: 
Variable containing:
  949.9868
-1183.3439
 1207.7928
[torch.FloatTensor of size 3]
```

调用backward()
```
In[47]: gradients = torch.FloatTensor([0.1, 1.0, 0.0001])
In[48]: y.backward(gradients)
In[49]: x.grad
Out[49]: 
Variable containing:
  102.4000
 1024.0000
    0.1024
[torch.FloatTensor of size 3]
```
