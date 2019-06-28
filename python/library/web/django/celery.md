---
title: Django Celery
date: 2019-06-28 15:36:00
tags: [django]
---

# celery
[celery](http://docs.celeryproject.org/en/latest/index.html)是一个任务队列，主要用于任务调度和定时任务。

# Django Celery
以前版本的Celery需要一个单独的库(django-celery)来和Djnago协同工作，新版本的[Celery](http://docs.celeryproject.org/en/latest/django/first-steps-with-django.html)可以直接在Django项目中使用。

## 一些准备工作
在`proj/proj/celery.py`文件中创建celery实例。
```
from __future__ import absolute_import, unicode_literals
import os
from celery import Celery

os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'djgraph.settings')

app = Celery('djgraph')
app.config_from_object('django.conf:settings', namespace='CELERY')
app.autodiscover_tasks()
```

在`proj/proj/__init__.py`中导入。
```
from __future__ import absolute_import, unicode_literals

from .celery import app as celery_app

__all__ = ('celery_app',)
```

在`setting.py`指定一些参数
```
CELERY_BROKER_URL = 'redis://192.168.0.201:6379/0'
CELERY_ACCEPT_CONTENT = ['json']
CELERY_TASK_SERIALIZER = 'json'
CELERY_RESULT_BACKEND = 'redis://192.168.0.201:6379/1'
```

## 创建一些任务
`Celery`会自动从app的`tasks.py`中寻找任务，只需要在`app/tasks.py`中用`shared_task`装饰器标记任务即可。
```
from celery import shared_task

@shared_task
def add(x, y):
    return x + y
```

## 一些定时任务
如果希望一些任务定时执行，可以在`settings.py`文件指定。
```
CELERY_BEAT_SCHEDULE = {
    'test': {
        'task': 'kanna.tasks.my_job',
        'schedule': 5,
        'args': ()
    }
}
```

## 启动
启动`worker`进程
linux:
```
celery -A proj worker -l info
```
windows:
```
celery -A proj worker --pool=solo -l info
```
启动`beat`服务，windows和linux相同
```
celery -A proj beat -l info
```

# Django Celery beat
[django-celery-beat](https://django-celery-beat.readthedocs.io/en/latest/index.html)扩展可以使用数据库管理定时任务。

## 创建一个定时任务
创建一个定时任务需要给出`schedule`和`task`。

`schedule`可以是`SolarSchedule`(日出、日落等太阳相关时间)、`ClockedSchedule`(一个精确的日期)、`IntervalSchedule`(固定间隔时间)、`CrontabSchedule`(crontab定时)。

`task`是在app中定义的任务。

例如创建一个每天12：00执行的定时任务。
```
schedule, created = CrontabSchedule.objects.get_or_create(
    minute='0',
    hour='12',
    timezone='Asia/Shanghai'
)
PeriodicTask.objects.create(
    crontab=schedule,
    name='add',
    task='kanna.tasks.start_job',
    args='[1, 2]'
)
```

## 启动
启动beat服务
```
celery -A proj beat -l info --scheduler django_celery_beat.schedulers:DatabaseScheduler
```
启动`worker`进程
windows:
```
celery -A djgraph worker --pool=solo -l info
```
linux:
```
celery -A djgraph worker -l info
```
