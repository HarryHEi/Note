---
title: myDjango-2 表单、路由、静态文件
date: 2016-05-13 08:59:02
tags: [django]
---

# Request 对象
## Request 对象包含的信息
HttpRequest对象包含请求URL的一些信息
```
request.path    # 除了域名以外的请求路径
request.get_host()    # 主机名
request.get_full_path()    # 包含查询字符串
request.is_secure()    # 是否通过https访问

# request.META是一个普通字典，如果键不存在，会返回KeyError异常
request.META['HTTP_REFERER']    # 进站前链接网页
request.META['HTTP_USER_AGENT']    # 用户浏览器user-agent字符串
request.META['REMOTE_ADDR']    # 客户端IP
```

## 提交数据信息
request.GET 和 request.POST 都是类字典对象，通过它们可以访问GET和POST数据

# 表单处理
## 使用GET
### 表单
表单中定义action为search，重定向到search视图函数
文件：**/myDjango/myDjango/templates/search_pic.html**
```
{% block content %}
<form action="/search/" method='get'>
    <input type='text' name='cont'>
    <input type='submit' value='Search'>
</form>
{% endblock %}
```
### 视图函数
search_pic是表单页面的视图
search视图函数从URL中获得GET参数
文件：**/myDjango/myDjango/views.py**
```
def search_pic(request):
    return render_to_response("search_pic.html")

def search(request):
    if 'cont' in request.GET:
        message = 'You searched for: %r' % request.GET['cont']
    else:
        message = 'You submitted an empty form.'
    return HttpResponse(message)
```
### 路由
针对两个视图函数定义两个路由
文件：**/myDjango/myDjango/urls.py**
```
from django.conf.urls import patterns, url, include

from myDjango.views import hello, current_datetime, hours_ahead, search_pic, search
from toon.views import showpic
from django.contrib import admin

urlpatterns = [
    #...
    url(r'^search-pic/$', search_pic),
    url(r'^search/$', search),
]
```
## 使用POST
### 表单
action改为''，post是在本页提交，URL不会发生变化

csrf_token用来防止跨站请求攻击
文件：**/myDjango/myDjango/templates/search_pic.html**
```
{% block content %}
<form action="" method='post'>
    {% csrf_token %}
    <input type='text' name='cont'>
    <input type='submit' value='Search' name="submit">
</form>
{% endblock %}
```
### 视图函数
search_pic是表单页面的视图
和之前类似，不过这次是POST数据

HttpResponseRedirect把页面重定向到某个页面
这里不能使用render_to_render来输出，因为可能造成数据的重复
文件：**/myDjango/myDjango/views.py**
```
from django.http import HttpResponseRedirect
from django.shortcuts import render_to_response

def search_pic(request):
    if request.POST.get('cont'):
        return HttpResponseRedirect("index.html")
    return render_to_response('search_pic.html')
```
如果需要传递参数，可以使用render
```
from django.shortcuts import render_to_response, render

def search_pic(request):
    if request.POST.get('cont'):
        print "ok"
        return render(request, "index.html",\
            {"label":"thanks"})
    return render_to_response('search_pic.html')
```
### 路由
只需要定义一个路由
文件：**/myDjango/myDjango/urls.py**
```
urlpatterns = [
    #...
    url(r'^search-pic/$', search_pic),
]
```

## Form 类 
Django带有一个form库，称为django.forms
使用这个库可以处理HTML表单及验证
文件：**/myDjango/myDjango/forms.py**
```
from django import forms

class testform(forms.Form):
    cont = forms.CharField()
```
在shell下操作
```
>>> from myDjango.forms import testform

>>> f=testform()
>>> print f
<tr><th><label for="id_cont">Cont:</label></th><td><input id="id_cont" name="cont" type="text" /></td></tr>

>>> print f['cont']
<input id="id_cont" name="cont" type="text" />

>>> f.errors
{}

>>> f['cont'].errors
[]

>>> f.is_valid()
False

>>> f=testform({"cont":"test"})
>>> f.is_valid()
True
```

### 视图函数
is_valid()验证过之后，从cleaned_data字典中获得参数
文件：**/myDjango/myDjango/views.py**
```
def search_pic(request):
    if request.method == 'POST':
        print request.POST
        form = testform(request.POST)
        if form.is_valid():
            d = form.cleaned_data['cont']
            print d
            return render(request, "index.html", {"label":d})
    else:
        form = testform()
    return render(request, 'search_pic.html', {"form":form})
```

### 渲染表单
使用form.as_table整齐排列
文件：**/myDjango/myDjango/templates/search_pic.html**
```
{% block content %}
<form action="" method='post'>
    {% csrf_token %}
    <table>
        {{ form.as_table }}
    </table>
    <input type='submit' value='Search' name="submit">
</form>
{% endblock %}
```

### 改字段显示
当指定一个字段时，Django使用默认Widget，也可以使用自定义Widget
即type="textarea"
文件：**/myDjango/myDjango/forms.py**
```
class testform(forms.Form):
    cont = forms.CharField()
    comment = forms.CharField(widget=forms.Textarea)
```

### 设置最大长度
同样的，最小长度为min_length
文件：**/myDjango/myDjango/forms.py**
```
class testform(forms.Form):
    cont = forms.CharField(max_length=100)
    comment = forms.CharField(widget=forms.Textarea)
```

### 初始值
在视图函数中传入
文件：**/myDjango/myDjango/views.py**
```
def search_pic(request):
    if request.method == 'POST':
        #...
    else:
        form = testform(initial={"cont":"lalala"})
    return render(request, 'search_pic.html', {"form":form})
```

### 自定义校验规则
Django的form系统自动寻找匹配函数的方法，该方法以clean_开头，以字段名称结束
如果有这样的方法，它将在默认校验结束后调用

下例在字数不足时会抛出ValidationError异常，追加到错误列表中
文件：**/myDjango/myDjango/forms.py**
```
class testform(forms.Form):
    cont = forms.CharField(max_length=100)
    comment = forms.CharField(widget=forms.Textarea)
    def clean_cont(self):
        cont = self.cleaned_data['cont']
        if len(cont) < 4:
            raise forms.ValidationError("not enough words")
        return cont
```
### 指定标签
HTML表单中自动生成的标签默认是按照规则生成的：用空格代替下划线，首字母大写。
也可以自定义标签，使用label
文件：**/myDjango/myDjango/forms.py**
```
class testform(forms.Form):
    cont = forms.CharField(max_length=100, label="cont")
    comment = forms.CharField(widget=forms.Textarea, label="com")
    #...
```

### Form设计
每个部件单独渲染，然后设置CSS样式
```
{% block content %}
<form action="" method='post'>
    {% csrf_token %}
    <div class="cont">
        {{ form.cont.errors }}
        <label for="id_cont">Cont</label>
        {{ form.cont }}
    </div>

    <div class="cont">
        {{ form.comment.errors }}
        <label for="id_comment">Comment</label>
        {{ form.comment }}
    </div>
    <input type='submit' value='Search' name="submit">
</form>
{% endblock %}
```

# 路由
为了防止在路由中导入视图函数的语句过长，可以导入视图文件本身
文件：**/myDjango/myDjango/urls.py**
```
from django.conf.urls import patterns, url, include

from myDjango import views
from toon.views import showpic
from django.contrib import admin

urlpatterns = [
    url(r'^admin/', include(admin.site.urls)),
    url(r'^$', views.hello),
    url(r'^time$', views.current_datetime),
    url(r'^time/plus/(\d+)/$', views.hours_ahead),
    url(r'^toon/',include('toon.urls')),
    url(r'^search-pic/$', views.search_pic),
]
```
Django一次匹配每个URL模式，直到匹配到第一个
成功捕获的URL以字符串形式传给视图函数

当Django找不到匹配请求的URL正则表达式时，会进入错误处理视图
```
handler404 —— django.conf.urls.handler404  django.views.defaults.page_not_found
handler500 —— django.conf.urls.handler500  django.views.defaults.server_error
handler403 —— django.conf.urls.handler403  django.views.defaults.permission_denied
handler400 —— django.conf.urls.handler400  django.views.defaults.bad_request
```

## 包含其他的URL
URL可以包含其他URL模块
```
urlpatterns = [
    url(r'^toon/',include(extra_patterns)),
]
extra_patterns = [
    url(r'^pic/', views.showpic),
]
```
或者写在里面
```
urlpatterns = [
    url(r'^some/', include([
        url(r'^history/$', views.history),
        url(r'^edit/$', views.edit),
        url(r'^discuss/$', views.discuss),
        url(r'^permissions/$', views.permissions),
    ])),
]
```

## 捕获参数
路由
```
urlpatterns = [
    #...
    url(r'^hello/(?P<name>\w+)$', views.hello),
]
```
视图
```
def hello(request, name="none"):
    return render_to_response("index.html",{"label":name})
```
参数的嵌套
匹配一个/hello/name=<name>格式的URL
当URL为 /hello/name=harryx 时，传入一个值为harryx的name参数给视图函数
```
urlpatterns = [
    #..
    url(r'^hello/(?:name=(?P<name>\w+)/)?$', views.hello),
]
```
路由传递参数
url函数有可选的第三个参数，以字典形式传递参数
```
urlpatterns = [
    #...
    url(r'^hello/(?:name=(?P<name>\w+)/)?$', views.hello, {"test":"lalala"}),
]
```
视图函数接收参数
```
def hello(request, name="none", test="test"):
    return render_to_response("index.html",\
        {"label":name+test})
```

## URL反向解析
定义路由
把URL命名为"year"
```
urlpatterns = [
    #..
    url(r'^hello/([0-9]{4})/$', views.hello, name="year"),
]
```
在模板中使用url模板标签，反向解析地址
```
{% url "year" 2013 %}
```
在视图函数中使用reverse解析
```
from django.core.urlresolvers import reverse

reverse("year", args=(2016,))
```

### 带命名空间的URL名字
因为在Django项目中可能会有很多应用，可以在命名空间下命名
文件：**myDjango/myDjango/urls.py**
```
urlpatterns = [
    url(r'^toon/',include('toon.urls', namespace="toon")),
]
```
文件：**myDjango/toon/urls.py**
```
urlpatterns = [
    url(r'^pic/', views.showpic, name="pic"),
]
```
文件：**myDjango/myDjango/templates/toon/pic.html**
```
<a href="{% url "toon:pic" %}">test</a>
```

### 通用视图
路由
```
urlpatterns = [
    url(r'^$', views.IndexView.as_view(), name="index"),
]
```
视图
```
class IndexView(generic.ListView):
    template_name = 'toon/index.html'
    context_object_name = 'pic_list'

    def get_queryset(self):
        return Pic.objects.order_by('picname')
```
模板
```
{% for p in pic_list %}
    <li>{{ p.picname }}</li>
{% endfor %}
```

# 静态文件
这里把静态文件static放在根目录下
setting.py配置
```
STATIC_URL = '/static/'

STATICFILES_DIRS = (
    os.path.join(BASE_DIR, "static"),
)
```
在模板文件中导入
```
<head>
    {% load staticfiles %}
    <link rel="stylesheet" type="text/css" href="{% static 'myDjango/test.css' %}" />
</head>
```

# 结语
Django框架先了解到这边
