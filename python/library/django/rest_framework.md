---
title: Django Rest Framework
date: 2018-07-19 17:28:00
tags: [django]
---

[QuikStart](https://www.django-rest-framework.org/tutorial/quickstart/)

[Home](https://www.django-rest-framework.org/)

# 安装

在安装Mysql支持的`mysql-client`时，windows需要使用`whl`文件安装，[地址](https://www.lfd.uci.edu/~gohlke/pythonlibs/)

# 配置

`INSTALLED_APPS`添加`rest_framework`

# Serializer

`Serializer`负责序列化以及反序列化`json`数据，主要提供给`View`使用，类似`Form`。

## 直接继承serializers.Serializer

[示例](https://www.django-rest-framework.org/tutorial/1-serialization/)

定义`Field`，以及`create`、`update`等方法

`View`直接调用Snippet的各方法(`create`、 `update`等)

## 继承serializers.ModelSerializer

[示例](https://www.django-rest-framework.org/tutorial/1-serialization/)

将serializer和model绑定，在Meta中设置model和fields

# View

[示例](https://www.django-rest-framework.org/tutorial/1-serialization/)

路由指定调用，处理用户请求，作出响应。

Rest API 不含 CSRF，因此需要使用`csrf_exempt`装饰器。

## 直接使用函数

[示例](https://www.django-rest-framework.org/tutorial/1-serialization/)

直接使用函数，第一个参数是`request`，后面参数由路由传递，比如`pk`。

## 使用 api_view 装饰器

api_view函数接收一个序列参数，包含支持的方法，比如`GET`、`POST`，不在序列内的方法会被聚拒绝。

## 继承 APIView

[示例](https://www.django-rest-framework.org/tutorial/3-class-based-views/)

继承 `APIView`，定义`get`、`post`、`put`、`delete`等方法。

在路由中使用as_view()方法获得处理函数。

## 继承 mixins 和 generics.GenericAPIView

[示例](https://www.django-rest-framework.org/tutorial/3-class-based-views/)

设置`queryset`和`serializer_calss`属性

定义`get`、`post`等方法，执行继承的`list`、`create`等行为，对serializer类绑定的model执行操作。

## 继承特殊generics

[示例](https://www.django-rest-framework.org/tutorial/3-class-based-views/)

继承特殊情况的`generics`，比如`ListCreateAPIView`用于列表查询以及新建实例、`RetrieveUpdateDestoryAPIView`用于查询、更新、删除实例

同样也是设置`queryset`和`serializer_calss`属性

也可以自定义`get`、`post`等方法。

## 继承 ViewSets

[示例](https://www.django-rest-framework.org/tutorial/6-viewsets-and-routers/)

继承ViewSets可以用来生成`list`、`post`、 `retrieve`等处理函数，或者使用Routers注册，会自动生成路由。

# 权限
## 登录路由

`login`和`logout`路由的配置

```
urlpatterns += [
    url(r'^auth/', include('rest_framework.urls')),
]
```

## permissions

[示例](https://www.django-rest-framework.org/tutorial/4-authentication-and-permissions/)

在View中指定`permission_classed`属性，值为一个元组，`permissions.IsAuthenticatedOrReadOnly`表示非登录只有只读权限

通过继承`permission.BasePermission`，自定义`has_object_permission`方法决定权限内容。
