---
title: 打印日志
date: 2017-04-01 15:00:20
tags: [python]
---

logging模块用于python的日志记录



不进行配置时，默认日志级别为warning

```
import logging
logging.debug("1")
logging.info("2")
logging.warning("3")
logging.error("4")
logging.critical("5")
```

输出内容：

```
WARNING:root:3
ERROR:root:4
CRITICAL:root:5
```

---

可以通过`logging.basicConfig`更改配置:

- level: 日志级别（DEBUG、INFO、WARNING、ERROR、CRITICAL）
- format: 输出格式
	- %(levelno)s: 日志级别数值
	- %(levelname)s: 日志级别名称
	- %(pathname)s: 执行的程序路径
	- %(filename)s: 执行文件名
	- %(funcName)s: 打印日志的函数
	- %(lineno)d: 行号
	- %(asctime)s: 时间
	- %(thread)d: 线程ID
	- %(threadName)s: 线程名称
	- %(process)d: 进程ID
	- %(message)s: 日志信息
- datefmt: 时间格式（%Y %b %d %a %H %M %S）（年、月、日、星期、时、分、秒）
- filename: 日志文件名
- filemode: 'w'或者'a'

```
logging.basicConfig(
    level=logging.DEBUG,
    format='%(asctime)s.%(filename)s[line:%(lineno)d] %(levelname)s %(message)s',
    datefmt='%a, %d, %b, %Y, %H:%M:%S',
    filename=os.path.join(os.path.dirname(__file__),
                          time.strftime('%Y-%m-%d', time.localtime())),
    filemode='w'
)
```

---

输出到屏幕

定义一个StreamHandler，将日志打印到屏幕，添加到当前日志处理对象。

```
logging_handler = logging.StreamHandler()
logging_handler.setLevel(logging.DEBUG)
logging_handler.setFormatter(logging.Formatter(
    '%(asctime)s.%(filename)s[line:%(lineno)d] %(levelname)s %(message)s'))
logging.getLogger('').addHandler(logging_handler)
```

---

日志回滚

限制每个日志文件大小最多1K（1024），最多备份五个日志文件，命名为my.log、my.log1~5，实际上一共6个文件，最新的为my.log，最旧的为my.log5

```
logging_handler2 = logging.handlers.RotatingFileHandler('my.log', maxBytes=1024, backupCount=5)
logging_handler2.setLevel(logging.DEBUG)
logging_handler2.setFormatter(logging.Formatter(
    '%(asctime)s.%(filename)s[line:%(lineno)d] %(levelname)s %(message)s'
))
logging.getLogger('').addHandler(logging_handler2)
```

常用配置
```
if not os.path.exists(os.path.join(os.path.dirname(__file__), 'log')):
    os.makedirs(os.path.join(os.path.dirname(__file__), 'log'))

root_logger = logging.getLogger('')
root_logger.setLevel(logging.INFO)

log_format = logging.Formatter('%(asctime)s.%(filename)s[line:%(lineno)d] %(levelname)s %(message)s')

stream_handler = logging.StreamHandler()
stream_handler.setFormatter(log_format)
root_logger.addHandler(stream_handler)

logging_handler = logging.handlers.RotatingFileHandler(
    os.path.join(os.path.dirname(__file__), 'log', 'log.txt'),
    maxBytes=10240000,
    backupCount=5,
    encoding='UTF-8'
)
logging_handler.setLevel(logging.INFO)
logging_handler.setFormatter(log_format)
root_logger.addHandler(logging_handler)
```

[参考](http://www.cnblogs.com/dkblog/archive/2011/08/26/2155018.html)
