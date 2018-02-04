[文档](https://docs.python.org/3/reference/datamodel.html)

# 特殊方法名
## 属性接口

属性接口用于获取类实例的属性。

### `object.__getattr__(self, name)`

当通过通常的方式没有找到属性时(即不是实例的属性也不是类属性)调用。

### `object.__getattribute__(self, name)`

当需要获取参数时无条件执行。
如果同时定义了`__getattr__()`和`__getattribute__()`，除非`__getattribute__()`显式指定或者抛出`AttributeError`，否则`__getattr__()`将不会被调用。
