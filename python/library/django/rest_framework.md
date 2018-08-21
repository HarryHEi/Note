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

```
INSTALLED_APPS = [
    ...
    'rest_framework',
    'rest_framework.authtoken',
    'djoser',
    'communications.apps.CommunicationsConfig',
    'users.apps.UsersConfig',
    'groups.apps.GroupsConfig',
    'prices.apps.PricesConfig',
    'api_auth.apps.ApiAuthConfig',
]
```

指定使用mysql，在建库的时候指定charset。

```
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'bcfm',
        'USER': 'satncs',
        'PASSWORD': 'satncs',
        'HOST': '192.168.0.201',
        'PORT': '3306',
        'OPTIONS': {
            'charset': 'utf8mb4',
        }
    }
}
```

使用JWT token

```
REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': (
        'rest_framework_jwt.authentication.JSONWebTokenAuthentication',
    ),
}
```

配置 JWT

```
JWT_AUTH = {
    'JWT_AUTH_COOKIE': 'token',  # 使用cookie
    'JWT_EXPIRATION_DELTA': datetime.timedelta(minutes=30),
    'JWT_ALLOW_REFRESH': True,
}
```

# Serializer

`Serializer`负责序列化以及反序列化`json`数据，主要提供给`View`使用，类似`Form`。

## 直接继承serializers.Serializer

[示例](https://www.django-rest-framework.org/tutorial/1-serialization/)

定义`Field`，以及`create`、`update`等方法

`View`直接调用Snippet的各方法(`create`、 `update`等)

## 继承serializers.ModelSerializer

[示例](https://www.django-rest-framework.org/tutorial/1-serialization/)

将serializer和model绑定，在Meta中设置model和fields

### 只读字段

如果在model是个function而不是一个真实字段，默认就是只读字段。

如果需要指定某个fields的字段为只读，同时添加这个字段到`read_only_fields`中。

### 自定义构造返回数据的方式

通过重写`to_representation`方法，可以自定义构造的数据的结构。

比如单独取出某个关系表的一个字段。
```
def to_representation(self, instance):
    data = super(UserSerializer, self).to_representation(instance)
    data.update({'groups': [group.name for group in instance.groups.all()]})
    device_serializer = DeviceSerializer(instance.devices, many=True)
    data.update({'devices': device_serializer.data})

    return data
```

或者构建层层嵌套的数据
```
def to_representation(self, instance):
    data = super(DeptSerializer, self).to_representation(instance)
    data.update(
        {
            'stations': [StationSerializer(station).data for station in instance.stations.all()]
        }
    )
    return data
```

### 自定义某个字段的校验

通过定义`validate_{字段值}`方法，对字段进行校验，如果校验不通过，可以抛出`ValidationError`异常。

### 自定义同时校验多个字段的方法

通过重写`to_internal_value`方法，该方法有一个`data`参数，是个字典，校验方法和校验一个字段类似，返回的也是字典。

### 自定义save方法

通过自定义save方法完成特定的保存操作，比如调用`set_password`设置用户的密码，而不是直接赋值。
```
def save(self, **kwargs):
    data = self.validated_data
    user = User.objects.get(username=data["user"]['username'])
    user.set_password(user.username)
    user.save()
```

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

### 自定义特殊方法

通过使用`action`装饰器，定义特殊方法，`methods`参数指定支持的HTTP方法，`detail`指定是否是指定某列，`permission_classes`可以特殊指定这个方法的权限验证类。

### 自定义表的查询方式

如果需要自定义表的查询方式，可以重写`get_queryset`方法，而不用指定`queryset`属性，但是如果使用router，需要在register时指定第三个参数`base_name`，一般为库名称的小写格式。

# 权限
## 登录路由

`login`和`logout`路由的配置

```
urlpatterns += [
    url(r'^auth/', include('rest_framework.urls')),
]
```

### JWT支持

`rest_framework_jwt`提供了JWT的获取以及刷新的View，添加到url。
```
from rest_framework_jwt.views import obtain_jwt_token, refresh_jwt_token

urlpatterns = [
    path('login/', obtain_jwt_token),
    path('refresh/', refresh_jwt_token),
]
```

## permissions

[示例](https://www.django-rest-framework.org/tutorial/4-authentication-and-permissions/)

在View中指定`permission_classed`属性，值为一个元组，`permissions.IsAuthenticatedOrReadOnly`表示非登录只有只读权限

通过继承`permission.BasePermission`，自定义`has_object_permission`方法决定权限内容。

```
class IsAdminOrOwnerReadOnly(BasePermission):
    """
    管理员或者是用户做自己账户相关的只读动作
    """

    def has_permission(self, request, view):
        return (
            view.detail and
            request.user == view.get_object() and
            request.method in SAFE_METHODS or
            request.user.is_authenticated and
            request.user.is_staff
        )


class IsAdminOrOwner(BasePermission):
    """
    管理员或者用户自己
    """

    def has_permission(self, request, view):
        return (
            view.detail and
            request.user == view.get_object() or
            request.user.is_authenticated and
            request.user.is_staff
        )

```
