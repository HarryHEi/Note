---
title: myDjango-0 基础
date: 2016-05-08 18:58:41
tags: [django]
---

# 简述
Django是python下的另一款服务器网络框架，属于重量级框架
MVC架构指的是：模型(model)，视图(views)，控制(controller)
由于Django自行处理控制部分即C，所以Django也被称为MTV框架，这里T指的是模板(Template)

# 项目结构
可以直接使用命令创建项目
```
django-admin.py startproject myDjango
```
创建的项目结构如下
```
myDjango
├── manage.py
└── myDjango
    ├── __init__.py
    ├── settings.py
    ├── urls.py
    └── wsgi.py
```
启动测试
```
python manage.py runserver 5000
```
# 视图和URL配置(路由)
一个简单的视图
文件：**myDjango/myDjango/views.py**
```
#导入HttpResponse类
from django.http import HttpResponse

def hello(request):
    return HttpResponse("hello world")
```
默认的urls.py注释了很多常用功能

url()除了常用的两个参数外，还有kwargs和name参数
kwargs参数是一个字典形式参数，用来传递参数给视图函数
name参数据是URL的名字

文件：**myDjango/myDjango/urls.py**
```
from django.conf.urls import patterns, url

#第一个元素是模式匹配字符串(正则表达式)
#第二个元素是视图函数
#这里^$分别匹配开头和结尾
urlpatterns = [
    url(r'^$', 'myDjango.views.hello'),
]
```

## render()
render()第一个参数是请求对象，第二个参数是模板名字，第三个参数是字典
返回一个HttpResponse对象
```
return render(request, "index.html", c)
```

## 引发一个404错误
```
from django.http import Http404

raise Http404("Not found")
```

## import
可以在路由开头使用import导入视图函数
```
from django.conf.urls import patterns, url
from myDjango.views import hello, current_datetime

urlpatterns = [
    url(r'^$', hello),
    url(r'^time$',current_datetime)
]
```
添加一个显示时间的视图函数
注意在显示中文时要在开头指定编码格式
文件：**myDjango/myDjango/views.py**
```
#!/usr/bin/env python
#!-*-coding:utf-8-*-

from django.http import HttpResponse
import datetime

def hello(request):
    return HttpResponse("hello world")

def current_datetime(request):
    now = datetime.datetime.now()
    html = "<html><body>现在时间:%s</body></html>" %now
    return HttpResponse(html)
```
时区是在setting.py中设置的
```
TIME_ZONE = 'Asia/Shanghai'
```
使用通配符处理URL，这里是匹配1位数及以上的数字
**如果作为参数传递，需要使用圆括号括起来**
文件：**myDjango/myDjango/urls.py**
```
url(r'^time/plus/(\d+)/$',hours_ahead)
```
视图函数需要接收传递的参数
request是一个HttpRequest对象，每个视图都是以它为第一个参数
文件：**myDjango/myDjango/views.py**
```
def hours_ahead(request, offset):
    try:
        offset = int(offset)
    except ValueError:
        raise Http404()
    now = datetime.datetime.now() + datetime.timedelta(hours=offset)
    html = "<html><body>现在时间:%s</body></html>" %now
    return HttpResponse(html)
```
# T 模板(Template)
Django的模板指的是文本，用于产生HTML
可以在HTML中写python
变量的写法：
```
{{ something }}
```
if语句和for循环
```
{% if a %}
    something
{% else %}
    another
{% endif %}

{% for a in as %}
    do something
{% endif %}
```
注释
```
{# 这是注释 #}
```
反向迭代
```
{% for a in as reversed %}
    do something
{% endif %}
```
过滤器
```
{{ name|lower }}
```
创建一个template实例
```
>>> from django.template import Template
>>> t=Template("i am {{ name }}")
```
可以使用context来传递参数给这个template对象
context在Django中是一个Context(上下文)对象
template对象的render()方法可以用来接收传递参数
```
>>> from django.template import Context
>>> c=Context({"name":"harry"})
>>> t.render(c)
u'i am harry'
>>> print t.render(c)
i am harry
```
同一个模板对象可以渲染多个context
```
>>> c2=Context({"name":"herui"})
>>> print t.render(c2)
i am herui
```
## 导入模板
配置
BACKEND 是一个指向Django后端API，内置的有
django.template.backends.django.DjangoTemplates
django.template.backends.jinja2.Jinja2 (Jinja2引擎)
必须使用绝对路径
文件：**myDjango/myDjango/setting.py**
```
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [os.path.join(os.path.dirname(__file__),"templates")],
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
            ],
        },
    },
]
```
视图函数
django.template.loader定义了两个函数加载模板

使用get_template()加载模板，返回一个Template对象
如果模板文件在templates子目录中，添相应的斜杠，get_template("west/example.html")
获得Template对象之后，使用Context渲染
文件：**myDjango/myDjango/views.py**
```
from django.http import HttpResponse
from django.template.loader import get_template
from django.template import Context

def hello(request):
    return HttpResponse(get_template("index.html")\
        .render({"label":"hello"}))
```
select_template()用模板名称列表作为参数
```
select_template(['story_%s_detail.html' % story.id, 'story_detail.html'])
```

简单的模板
文件：**myDjango/myDjango/templates/index.html**
```
<h2>{{ label }}</h2>
```
### render_to_response()
render_to_response()能直接实现载入模板并渲染

+ 第一个参数是使用的模板名称
+ 第二个参数是Context字典，可选参数

文件：**myDjango/myDjango/views.py**
```
from django.shortcuts import render_to_response

def hello(request):
    return render_to_response("index.html",\
        {"label":"hello world"})
```
### locals()
locals()包括了到执行时间点所有变量的映射
这里的locals()就相当于之前的字典
locals()的值为
```
{
    'request': <WSGIRequest: GET '/time'>, 
    'label': datetime.datetime(2016, 5, 11, 16, 34, 19, 157325)
}
```
文件：**myDjango/myDjango/views.py**
```
def current_datetime(request):
    label = datetime.datetime.now()
    return render_to_response("index.html",locals())
```
### include
在模板文件中可以导入其他模板
```
{% include "index2.html" %}
{% include "ano/index3.html" %}
{% include temp_name %}
```
在index中导入include/time.html
如果在debug模式，没找到导入的文件会报错
如果不在debug模式，会无视这个导入
文件：**myDjango/myDjango/templates/include/time.html**
```
<h2>{{ current_time }}</h2>
```
文件：**myDjango/myDjango/templates/index.html**
```
{% include "include/time.html" %}
<h2>{{ label }}</h2>
```
视图函数
访问页面显示时间和问候语
```
from django.shortcuts import render_to_response
import datetime

def current_datetime(request):
    current_time = datetime.datetime.now()
    label = "hello"
    return render_to_response("index.html",locals())
```
## 模板继承
先构造一个基础模板，然后子模板对块进行修改
基础模板中把可以更改的地方用block块包起来
文件：**myDjango/myDjango/templates/base.html**
```
<!DOCTYPE html>
<html lang="en">
    {% block head %}
    <head>
        <meta charset=utf-8 / >
        <title>{% block title %}myDjango{% endblock %}</title>
    </head>
    {% endblock %}

    {% block body %}
    <body>
        <h1>welcome to my Django</h1>
        {% block content %}{% endblock %}
        <p>thanks</p>
    </body>
    {% endblock %}
</html>
```
使用extends继承
```
{% extends "base.html" %}

{% block title %}Home{% endblock %}

{% block content %}
<p>lalalala</p>
{% endblock %}
```

[参考](http://python.usyiyi.cn/django/index.html)
