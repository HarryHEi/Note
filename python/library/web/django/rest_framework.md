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

对于没有保存到数据库的数据，当需要序列化对象时可以继承`Serializer`


例如：

```
class ConfigureSerializer(Serializer):
    root = BooleanField()
    name = CharField(max_length=16)
    visit_address = CharField(max_length=32)
    higher_host = CharField(max_length=32, default='')

    def create(self, validated_data):
        raise ValidationError('Not Support create')

    def update(self, instance, validated_data):
        raise ValidationError('Not Support update')

    def to_representation(self, instance):
        return super(ConfigureSerializer, self).to_representation(instance)

    def to_internal_value(self, data):
        return {
            'root': data.get('root'),
            'name': data.get('name'),
            'visit_address': data.get('visit_address'),
            'higher_host': data.get('higher_host', ''),
        }
```

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

通过重写`to_internal_value`方法，该方法有一个`data`参数，是个字典，校验方法和校验一个字段类似，返回的也是字典。可以先调用继承来的`to_internal_value`，然后再校验
```
def to_internal_value(self, data):
    value = super(ControlChannelSerializer, self).to_internal_value(data)

    freq_segment = value['freq_segment']
    up_freq_begin = value['up_freq_begin']
    up_freq_end = value['up_freq_end']
    down_freq_begin = value['down_freq_begin']
    down_freq_end = value['down_freq_end']
    
    if up_freq_end - up_freq_begin != down_freq_end - down_freq_begin:
        raise ValidationError(
            {
                'error': 'up_freq_end - up_freq_begin == down_freq_end - down_freq_begin'
            }
        )

    return value
```

但是这样会有问题，如果是使用patch方式调用接口，可能并不含有所有字段，因此要加上默认值，可以从instance中获取当前值，创建的时候是没有instance的，所以加上`if self.instance`判断
```
def to_internal_value(self, data):
    value = super(ControlChannelSerializer, self).to_internal_value(data)

    freq_segment = value.get('freq_segment', self.instance.freq_segment if self.instance else None)
    up_freq_begin = value.get('up_freq_begin', self.instance.freq_segment.up_freq_begin if self.instance else None)
    up_freq_end = value.get('up_freq_end', self.instance.up_freq_end if self.instance else None)
    down_freq_begin = value.get('down_freq_begin', self.instance.down_freq_begin if self.instance else None)
    down_freq_end = value.get('down_freq_end', self.instance.down_freq_end if self.instance else None)

    if up_freq_end - up_freq_begin != down_freq_end - down_freq_begin:
        raise ValidationError(
            {
                'up_freq_end': 'up_freq_end - up_freq_begin == down_freq_end - down_freq_begin',
                'up_freq_begin': 'up_freq_end - up_freq_begin == down_freq_end - down_freq_begin',
                'down_freq_end': 'up_freq_end - up_freq_begin == down_freq_end - down_freq_begin',
                'down_freq_begin': 'up_freq_end - up_freq_begin == down_freq_end - down_freq_begin',
            }
        )

    return value
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

针对没有数据库的接口，且提交的字段不定，通过继承`APIView`可以很方便的实现自定义的接口。

比如实现一个代理

```
def _requests_request(method, url, *_, **kwargs):
    """
    发起HTTP请求
    :param method: str 'get' 'post' and so on
    :return: status, result
    """
    try:
        res = requests.request(method, url, **kwargs)
    except requests.exceptions.ConnectionError:
        return HTTP_500_INTERNAL_SERVER_ERROR, '{} ConnectionError'.format(url)
    try:
        data = json.loads(res.content)
    except json.decoder.JSONDecodeError:
        data = res.text
    return res.status_code, data

class ApiGetViewMixin:
    def get(self, request):
        url = self._get_url()
        status, result = _requests_request('get', url)
        return Response(data=result, status=status)

    def _get_url(self):
        return None


class UsingControlChannelView(ApiGetViewMixin, APIView):
    def _get_url(self):
        return _make_using_control_channel_url()
```

## 继承 mixins 和 generics.GenericAPIView

[示例](https://www.django-rest-framework.org/tutorial/3-class-based-views/)

设置`queryset`和`serializer_calss`属性

定义`get`、`post`等方法，执行继承的`list`、`create`等行为，对serializer类绑定的model执行操作。

针对没有数据库，但是请求的字段是固定的情况，可以继承`generics.GenericAPIView`

例如:

```
class ConfigureView(generics.GenericAPIView):
    """
    分级管理器配置
    """
    serializer_class = ConfigureSerializer
    permission_classes = [IsAdminOrReadOnly]

    def get(self, request):
        return do_proxy_get(LTL_CONFIGURE_URL)

    def post(self, request):
        serializer = self.get_serializer(data=request.data)
        if serializer.is_valid():
            return do_proxy_post(LTL_CONFIGURE_URL, serializer.data)
        else:
            return Response(data=serializer.errors, status=HTTP_400_BAD_REQUEST)
```

## 继承特殊generics

[示例](https://www.django-rest-framework.org/tutorial/3-class-based-views/)

继承特殊情况的`generics`，比如`ListCreateAPIView`用于列表查询以及新建实例、`RetrieveUpdateDestoryAPIView`用于查询、更新、删除实例

同样也是设置`queryset`和`serializer_calss`属性

也可以自定义`get`、`post`等方法。

## 继承 ViewSets

[示例](https://www.django-rest-framework.org/tutorial/6-viewsets-and-routers/)

继承ViewSets可以用来生成`list`、`post`、 `retrieve`等处理函数，或者使用Routers注册，会自动生成路由。

### 自定义表的查询方式

如果需要自定义表的查询方式，可以重写`get_queryset`方法，而不用指定`queryset`属性，但是如果使用router，需要在register时指定第三个参数`base_name`，一般为库名称的小写格式。


## 分页
### 使用游标
游标使用的Pagination是`CursorPagination`，查询的API形如`?cursor=**`
cursor的值为base64编码的`r=*&p=*`，查询结果形如
```
HTTP 200 OK
{
    "next": "http://127.0.0.1:8888/api/cw/messages/?cursor=cD0xNjY%3D",
    "previous": "http://127.0.0.1:8888/api/cw/messages/?cursor=cj0xJnA9MTcw",
    "results": [
        ...
    ]
}
```
继承`CursorPagination`，然后设置`ordering`属性，标识排序指定的字段
```
class MessageStandardPagination(CursorPagination):
    """
    查询字符串中包含 p=?,
    服务器会检索id为p之后的内容，不包括id为p
    """
    page_size = 5
    ordering = "-id"
```
最后在view中指定分页类
```
class MessageViewSet(CreateListRetrieveDestroyViewSet):
    queryset = Message.objects.all()
    serializer_class = MessageSerializer
    pagination_class = MessageStandardPagination
    permission_classes = [IsAuthenticated]
```

# 用户认证
## model

当需要在用户中添加其他额外字段时，需要使用`OneToOneField`建立一个新表。

```
class Info(models.Model):
    user = models.OneToOneField(User, on_delete=models.CASCADE)
    phone_number = models.CharField(max_length=11, unique=True)
    remark = models.CharField(max_length=32, default='', blank=True)

    def __repr__(self):
        return '<{}>'.format(str(self))

    def __str__(self):
        return 'Info({})'.format(self.user.username)
```

## serializer

因为添加了新的字段，所以需要在创建用户时提供额外字段，同时也要建立一个新表的Serializer，为了方便验证。

```
class InfoSerializer(ModelSerializer):
    class Meta:
        model = Info
        fields = '__all__'


class UserSerializer(ModelSerializer):
    phone_number = CharField(source='info.phone_number')
    remark = CharField(source='info.remark', default='', allow_blank=True)

    class Meta:
        model = User
        fields = ('id', 'username', 'phone_number', 'is_staff', 'password', 'remark')
        read_only_fields = ('devices', 'is_staff')

    def to_representation(self, instance):
        data = super(UserSerializer, self).to_representation(instance)
        # 过滤掉password
        del data['password']

        return data

    def to_internal_value(self, data):
        value = super(UserSerializer, self).to_internal_value(data)
        # 去掉info字段，后面单独处理
        value['info'] = None
        return value
```

## view

当写View时，需要考虑两个表的新增和更新。

```
class UserViewSet(ModelViewSet):
    """管理用户"""

    queryset = User.objects.all()
    serializer_class = UserSerializer

    def perform_create(self, serializer):
        user = serializer.save()
        request_data = self.request.data
        user.set_password(request_data['password'])
        request_data['user'] = user.id
        info_serializer = InfoSerializer(data=request_data)
        try:
            info_serializer.is_valid(raise_exception=True)
        except Exception as e:
            user.delete()
            raise e
        info = info_serializer.save()
        user.info = info
        user.save()

    def perform_update(self, serializer):
        user = self.get_object()
        request_data = self.request.data
        info_serializer = InfoSerializer(user.info, data=request_data, partial=True)
        info_serializer.is_valid(raise_exception=True)
        info_serializer.save()
        user = serializer.save()
        if 'password' in request_data:
            user.set_password(request_data['password'])
            user.save()
```

可以把密码字段过滤掉，不再修改用户基本信息时修改。

比如建立独立的serilaizer和view

```
class ChangePasswordSerializer(Serializer):
    """修改密码"""

    old_password = CharField(max_length=16)
    new_password = CharField(max_length=16)

    def to_internal_value(self, data):
        old_password = data.get('old_password')
        new_password = data.get('new_password')

        return {
            'old_password': old_password,
            'new_password': new_password
        }

class UserChangePWD(GenericViewSet):
    queryset = User.objects.all()
    serializer_class = ChangePasswordSerializer

    @action(methods=['POST'], detail=True)
    def password(self, request, pk=None):
        """
        修改密码
        """
        serializer = ChangePasswordSerializer(data=request.data)
        if serializer.is_valid():
            data = serializer.validated_data
            user = self.get_object()
            if request.user.id != user.id and not request.user.is_staff:
                return Response(status=HTTP_403_FORBIDDEN)
            if authenticate(username=user.username, password=data['old_password']) is not None:
                user.set_password(data['new_password'])
                user.save()
                return Response(status=HTTP_204_NO_CONTENT)
            return Response(
                data={
                    'old_password': 'password is invalid'
                },
                status=HTTP_400_BAD_REQUEST
            )
        return Response(data=serializer.errors, status=HTTP_400_BAD_REQUEST)
```

如果使用了cookie来存储JWT，可以新增删除cookie的登出操作

```
class LogoutView(APIView):
    permission_classes = [AllowAny, ]

    def post(self, request, *args, **kwargs):
        response = Response(status=HTTP_204_NO_CONTENT)
        response.delete_cookie('token')
        return response

urlpatterns = [
    path('logout/', LogoutView.as_view(), name='jwt-auth-logout')
]

```

# Filters
## 自定义filter backend
自定义filter backend需要继承`rest_framework.filters.BaseFilterBackend`，重写`filter_queryset`，返回queryset

例如根据时间范围过滤日志
```
from rest_framework.filters import BaseFilterBackend
from rest_framework import compat

class TimeFilterBackend(BaseFilterBackend):
    """
    根据时间过滤

    time_filter_fields = ('time', )
    time_filter_fields = ('start_time', 'time')
    """
    @staticmethod
    def _make_query_params(filters, begin, end):
        """
        time_filter_fields = ('time', ) => {'time__gte': begin, 'time__lte': end}
        time_filter_fields = ('start_time', 'time') => {'start_time__gte': begin, 'time__lte': end}

        如果begin和end为None，则缺省该字段
        """
        result = {}
        if not filters:
            return result

        if len(filters) == 1:
            time = filters[0]
            if begin:
                result.update({'{}__gte'.format(time): begin})
            if end:
                result.update({'{}__lte'.format(time): end})
        else:
            start_time, end_time, *_others = filters
            if begin:
                result.update({'{}__gte'.format(start_time): begin})
            if end:
                result.update({'{}__lte'.format(end_time): end})

        return result

    def filter_queryset(self, request, queryset, view):
        time_filter_fields = getattr(view, 'time_filter_fields', None)
        if not time_filter_fields:
            return queryset

        begin, end = get_begin_end_from_request(request)
        query_params = self._make_query_params(time_filter_fields, begin, end)
        return queryset.filter(**query_params)

    def get_schema_fields(self, view):
        return [
            compat.coreapi.Field(
                name='begin',
                required=False,
                location='query',
                schema=compat.coreschema.String(description='')
            ),
            compat.coreapi.Field(
                name='end',
                required=False,
                location='query',
                schema=compat.coreschema.String(description='')
            )
        ]
```

使用方法
```
class CULogViewSet(ModelViewSet):
    queryset = CULog.objects.all()
    serializer_class = CULogSerializer
    filter_backends = (TimeFilterBackend, )
    time_filter_fields = ('time', )
```

`_make_query_params`是自定义方法，根据查询字段`begin`和`end`以及从view中获取的`time_filter_fields`生成查询条件

`filter_queryset`使用查询条件查询并返回query_set

`get_schema_fields`用于api docs接口显示，这里固定是`begin`和`end`字段。

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

# 测试用例
每隔测试用例是独立的上下文，编写测试用例时可以先创建一些函数用来快捷添加用户。
```
def create_common_user(username='test_user', password='user_password'):
    user = User.objects.create(username=username)
    user.set_password(password)
    user.save()
    return user

def common_user_login(client, username='test_user', password='user_password'):
    response = client.post(
        reverse('jwt-auth:jwt-auth-login'),
        data={
            'username': username,
            'password': password
        }
    )
    return response

def get_common_user_authorization(client, username='test_user', password='user_password'):
    response = common_user_login(client, username, password)
    return 'JWT {}'.format(response.data['token'])
```
需要模拟用户请求时，有时候需要添加用户认证信息，添加`HTTP_AUTHORIZATION`即可。
```
response = self.client.post(
    post_url,
    data={
        'username': 'test_user',
        'password': 'test_password'
    },
    HTTP_AUTHORIZATION=token
)
```

# 国际化支持
[文档](https://docs.djangoproject.com/en/2.1/topics/i18n/translation/)
## 配置
使用Django 的 `makemessages`命令时，需要在settings文件配置查找目录，这里为根目录下的`locale`目录。
```
LOCALE_PATHS = [
    os.path.join(BASE_DIR, 'locale')
]
```
## 使用
```
from django.utils.translation import ugettext_lazy as _

class SomeView(ApiView):
    def post(self, request):
        raise ValidationError({
            'freq': _('Not enough resources.')
        })
```

## 生成`.po`文件
```
python manage.py makemessages -l zh_Hans
```
生成的文件格式形如
```
msgid "Not enough resources."
msgstr ""
```
需要把对应语言的字符串写到msgstr
```
msgid "Not enough resources."
msgstr "没有足够的资源。"
```

## 编译成`.mo`文件
```
python manage.py compilemessages
```

# 部署
## python环境
项目文件拷贝到`/home/xxx`，切换到该目录。

安装virtualenv
```
sudo apt-get install virtualenv
```
安装pip3
```
sudo apt-get install python3-pip
```
创建虚拟环境
```
virtualenv env --python=python3
```
启用虚拟环境
```
source venv/bin/active
```

## 数据库
### 安装mysql

[安装](https://www.digitalocean.com/community/tutorials/how-to-install-mysql-on-ubuntu-16-04)
```
sudo apt-get install mysql-server
sudo apt-get install python-dev libmysqlclient-dev
mysql_secure_installation
```
如果使用比较简单的密码
```
sudo mysql -u root -p
uninstall plugin validate_password;
```

### 创建用户和数据库

[创建用户](https://www.digitalocean.com/community/tutorials/how-to-create-a-new-user-and-grant-permissions-in-mysql)
```
CREATE USER 'newuser'@'localhost' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON * . * TO 'newuser'@'localhost';
FLUSH PRIVILEGES;
```

创建库的时候指定编码为`utf8mb4`
```
create database xxx default charset = "utf8mb4";
```

### 修改setting文件
修改文件中指定的库地址

修改debug为False

修改白名单为本地ip

## 静态文件部署
将静态文件拷贝到`/usr/share/nginx/`;

## nginx
配置文件`nginx.conf`
```
upstream django {
    server unix:/home/admin/cwm/cwm.sock;
}

server {
    server_name localhost;

    location ~ ^/api/cwm {
        uwsgi_pass django;
        include uwsgi_params;
    }

    location / {
        root cwm;
        index index.html;
    }
}
```

将配置文件`nginx.conf`拷贝到`/etc/nginx/site-enabled/`;
```
sudo servcice nginx restart
```

## wsgi
安装
```
pip3 install uwsgi
```
配置
```
[uwsgi]
project = cwm
base = /home/admin

chdir = %(base)/%(project)
home = %(base)/%(project)/venv
module = %(project).wsgi:application

logto = %(base)/%(project)/logs/cwm.log

master = true
processes = 5

socket = %(base)/%(project)/%(project).sock
chmod-socket = 777
vacuum = true

```
启动
```
uwsgi --emperor conf/xxx.ini
```

## supervisor
安装
```
sudo apt-get install supervisor
```
配置
```
[program:cwm]
command=/home/satncs/cwm/venv/bin/uwsgi /home/satncs/cwm/conf/uwsgi.ini
directory=/home/satncs/cwm
user=satncs
autostart=true
autorestart=true
stdout_logfile=/home/satncs/cwm/logs/uwsgi_supervisor.log
```
启动
```
service supervisor start
```

## 使用key登录服务器

[参考](https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys--2)

生成公钥和私钥
```
ssh-keygen -t rsa
```

拷贝公钥到服务器
```
ssh-copy-id user@host
```

## 建立ssh tunnel

[参考](https://www.cnblogs.com/37yan/p/9003423.html)

使用远程转发的方式
```
ssh -o ServerAliveInterval 30 -o ServerAliveCountMax 3 -p port -NfR remote_host:remote_port:local_host:local_port remote_user@host
```

注意不能使用数据库的root账号登录。

如果希望自动重连，使用autossh代替ssh。

```
autossh -M 0 -o ServerAliveInterval 30 -o ServerAliveCountMax 3 -p port -NfR remote_host:remote_port:local_host:local_port remote_user@host
```

## 给与用于远程连接数据库的权限

如果需要的话，可以给用户远程登录数据库的权限

首先需要修改数据库监听的地址
```
vi /etc/mysql/mysql.conf.d/mysql.cnf

bind-address = 0.0.0.0
```

创建远程访问的用户，给与数据库的权限。
```
create user 'user'@'%' identified by 'password';
grant all on database_name.* to 'user'@'%';
```
