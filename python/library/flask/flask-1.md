---
title: myblog-01 开始
date: 2016-04-10 17:14:02
tags: [flask]
---

# 简述
根据《Flask Web开发：基于Python的Web应用开发实战》一书搭建自己的博客网站并运行在云服务器上
# 项目结构
```
|-myblog/ #根目录
    |- app/ #Flask程序
        |- auth/ #认证蓝本
            |- __init__.py #创建蓝本
            |- views.py #蓝本中的路由和视图函数
        |- templates/ #模板文件
        |- static/ #静态文件
        |- main/
            |- __init__.py #创建蓝本
            |- errors.py #错误处理程序
            |- forms.py #表单
            |- views.py #路由
        |- __init__.py #程序包构造文件，注册蓝本
        |- email.py
        |- models.py #数据库模型
    |- migrations/ #数据库迁移脚本
    |- tests/ #单元测试
        |- __init__.py
        |- test*.py
    |- venv/ #Python虚拟环境
    |- requirement.txt #列出所有依赖包
    |- config.py #储存配置
    |- manage.py #用于启动程序以及其他的程序任务
```
# 基础
## Flask类
Flask构造函数只有一个必须的参数，即程序主模块的名字
```
from flask import Flask
app = Flask(__name__)

#启动
app.run(debug=True)
```
## 视图函数
```
@app.route('/')
def index():
    return "<h1>hello</h1>"
```
## 命令解析器
```
from flask.ext.script import Manager
manager = Manager(app)

# 开始
manager.run()

$ python blog.py runserver
```
# 模板
## Jinja2模板引擎
```
#文件: templates/index.html
<h1>Hello</h1>

#文件: emplates/user.html
<h1>{{ name }}'s blog</h1>

# 导入Jinja2
from flask import render_template

...
return render_temptale('index.html')

...
return render_template('user.html', name=name)
```
## 变量
在模板中使用**{{ name }}**结构表示一个变量
用过滤器修改变量，{{ name|capitalize }}
## 控制语句
## 条件语句
```
{% if user %}
    hello, {{ user }}!
{% else %}
    hello, Stranger!s
{{% endif %}}
```
### for循环
```
<ul>
    {% for comment in comments %}
        <li>{{ comment }}</li>
    {% endfor %}
</ul>
```
## 模块继承
### templates/base.html文件
```
#模板文件：templates/base.html
#模板文件会预留出{{...}}块，子模板可以更改这些块

<html>
  <head>
    {% block head %}
    <title>{% block title %}{% endblock %} - My Blog</title> 
    {% endblock %}
  </head>
  <body>
    {% block body %}
    {% endblock %}
  </body>
</html>
```
### templates/index.html文件
```
#子模板文件：templates/index.html
#使用{% extends "base.html" %}继承模板，然后重写相应的块

{% extends "base.html" %}

{% block title %}Index{% endblock %}

{% block body %}
<h1>hello</h1>
{% endblock %}
```
## 使用Flask-Bootstrap集成Twitter Bootstrap
**初始化**Flask-Bootstrap
```
from flask.ext.bootstrap import Bootstrap
bootstrap = Bootstrap(app)
```
### templates/base.html文件
调用bootstrap
```
#从Flask-Bootstrap导入bootstrap/base.html
{% extends "bootstrap/base.html" %}

#页面标题
{% block title %}Flasky{% endblock %}

#导航条
{% block navbar %}
<div class="navbar navbar-inverse" role="navigation">
    <div class="container">
        <div class="navbar-header">
            <button type="button" class="navbar-toggle" data-toggle="collapse" data-target=".navbar-collapse">
                <span class="sr-only">Toggle navigation</span>
                <span class="icon-bar"></span>
                <span class="icon-bar"></span>
                <span class="icon-bar"></span>
            </button>
            <a class="navbar-brand" href="/">Flasky</a>
        </div>
        <div class="navbar-collapse collapse">
            <ul class="nav navbar-nav">
                <li><a href="/">Home</a></li>
            </ul>
        </div>
    </div>
</div>
{% endblock %}

{% block content %}
<div class="container">
    {% block page_content %}{% endblock %}
</div>
{% endblock %}
```
### templates/index.html文件
调用base.html，而不直接调用bootstrap
```
{% extends "base.html" %}

{% block title %}Index{% endblock %}

{% block page_content %}
<div class="page-header">
    <h1>Welcome to harry's blog</h1>
</div>
<p>some message</p>
{% endblock %}
```
### templates/user.html文件
```
{% extends "base.html" %}

{% block title %}Index{% endblock %}

{% block page_content %}
<div class="page-header">
    <h1>{{ name }}'s blog</h1>
</div>
<p>some message</p>
{% endblock %}
```
### 自定义错误页面
编写路由和视图函数
```
@app.errorhandler(404)
def page_not_found(e):
    return render_template('404.html'), 404
```
**templates/404.html**
```
{% extends "base.html" %}

{% block title %}Index{% endblock %}

{% block page_content %}
<div class="page-header">
    <h1>Not found</h1>
</div>
{% endblock %}
```
    
## super()
如果需要在已有的块中添加新内容，需要使用**super()**函数
使用**url_for()**函数，获得对应的URL
```
url_for('user', name='join', _external=True)
#返回:http://localhost:5000/user/join
```
在**templates/base.html**添加
```
{% block head %}
{{ super() }}
<link rel="icon" href="{{ url_for('static', filename='shortcut.ico') }}" type="image/x-icon">
{% endblock %}
```

[bootstrap中文网](http://v3.bootcss.com/)
[bootstrap示例](http://v3.bootcss.com/getting-started/)
