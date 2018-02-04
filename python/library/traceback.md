---
title: 异常处理
date: 2017-04-01 17:11:08
tags: [python]
---

python捕获异常

```
try:
    a = 1 / 0
except Exception, e:
    print str(e)
```

输出

```
integer division or modulo by zero
```

---

可以继承Exception类

```
class AIsZeroError(Exception):
    pass

try:
    raise AIsZeroError('a is 0')
except AIsZeroError, e:
    print str(e)
```

输出

```
a is 0
```

---

python中使用traceback模块处理异常，sys模块也提供了特定的参数和函数。

sys.exc_info()

该函数会返回一个三元元组，包含异常信息（exc_type exc_value exc_traceback）

```
try:
    a = 1 / 0
except Exception:
    t, value, tb = sys.exc_info()
    print t
    print value
    print tb
```

输出

```
<type 'exceptions.ZeroDivisionError'>
integer division or modulo by zero
<traceback object at 0x000000000367A5C8>
```

---

traceback.print_exception()

格式化输出异常

```
try:
    a = 1 / 0
except Exception:
    t, value, tb = sys.exc_info()
    traceback.print_exception(t, value, tb)
```

输出

```
Traceback (most recent call last):
  File "D:/MyData/code/python_test/just_fo_test/main.py", line 5, in <module>
    a = 1 / 0
ZeroDivisionError: integer division or modulo by zero
```

---

traceback.print_exc()

traceback.print_exc()函数只是traceback.print_exception()函数的一个简写形式

```
try:
    a = 1 / 0
except Exception:
    traceback.print_exc(file=sys.stdout)
```

---

traceback.format_exception()

传入参数，返回一个数组

```
try:
    a = 1 / 0
except Exception:
    t, value, tb = sys.exc_info()
    for l in traceback.format_exception(t, value, tb):
        print l
```

输出

```
Traceback (most recent call last):

  File "D:/MyData/code/python_test/just_fo_test/main.py", line 5, in <module>
    a = 1 / 0

ZeroDivisionError: integer division or modulo by zero
```
