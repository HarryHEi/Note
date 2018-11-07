---
title: myDjango-1 数据库
date: 2016-05-11 18:51:46
tags: [django]
---

# 添加一个功能(app)
命令行
```
python manage.py startapp toon
```
更改配置
文件：**myDjango/myDjango/settings.py**
```
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'toon',
]
```
# 添加模型
在之前添加app的命令执行后会创建一个模型文件
文件：**myDjango/myDjango/toon/models.py**
```
class Pic(models.Model):
    picname = models.CharField(max_length=32)
    pic = models.ImageField(null=True,blank=True)
    tips = models.TextField()
```
## 数据类型
```
1、models.AutoField　　自增列=int(11)
　 如果没有的话，默认会生成一个名称为id的列，如果要显示的定义一个自增列，必须把该列设置为主键(primary_key=True)
2、models.CharField　　字符串类型字段 必须加max_length参数
3、models.BooleanField　布尔类型字段=tinyint(1)  不能为空，Blank=True
4、models.ComaSeparatedIntegerField　　用逗号分割的数字类型=varchar 继承CharField，所以必须加max_lenght参数
5、models.DateField　　日期字段类型date
　 参数auto_now=True表示每次更新都会更新这个时间；参数auto_now_add表示只是第一次创建时添加，之后的更新不再改变
6、models.DateTimeField　　日期字段类型datetime  同DateField的参数
7、models.Decimal　　十进制小数类型=decimal
　 必须指定整数位max_digits和小数位decimal_places
8、models.EmailField　　字符串类型(正则表达式邮箱)=varchar  对字符串进行正则表达式验证
9、models.FloatField　　浮点类型=double
10、models.IntegerField　　整形
11、models.BigIntegerField　长整形
　　integer_field_ranges = {
　　　　'SmallIntegerField': (-32768, 32767),
　　　　'IntegerField': (-2147483648, 2147483647),
　　　　'BigIntegerField': (-9223372036854775808, 9223372036854775807),
　　　　'PositiveSmallIntegerField': (0, 32767),
　　　　'PositiveIntegerField': (0, 2147483647),
　　}
12、models.IPAddressField　　字符串类型(ip4正则表达式)
13、models.GenericIPAddressField　　字符串类型（ip4和ip6是可选的）
　　参数protocol可以是：both、ipv4、ipv6  验证时，会根据设置进行报错
14、models.NullBooleanField　　允许为空的布尔类型
15、models.PositiveIntegerFiel　　正Integer
16、models.PositiveSmallIntegerField　　正smallInteger
17、models.SlugField　　减号、下划线、字母、数字
18、models.SmallIntegerField　　数字
　　数据库中的字段有：tinyint、smallint、int、bigint
19、models.TextField　　字符串=longtext
20、models.TimeField　　时间 HH:MM[:ss[.uuuuuu]]
21、models.URLField　　字符串类型，地址正则表达式
22、models.BinaryField　二进制
23、models.ImageField   图片
24、models.FilePathField 文件
```
## 参数
```

1、null=True 数据库中字段是否可以为空
2、blank=True django的Admin中添加数据时是否可允许空值
3、primary_key=False 主键，对AutoField设置主键后，就会代替原来默认的自增id列
4、auto_now和auto_now_add
　　auto_now  自动创建---无论添加或修改，都是当前操作的时间
　　auto_now_add  自动创建---永远是创建时的时间
5、choices
     GENDER_CHOICE = (
          (u'M', u'Male'),
          (u'F', u'Female'),
      )
     gender = models.CharField(max_length=2,choices=GENDER_CHOICE)
6、max_length 最大长度
7、default　　默认值
8、verbose_name　　Admin中字段的显示名称
9、name|db_column　数据库中的字段名称
10、unique=True　　不允许重复
11、db_index = True　　数据库索引
12、editable=True　　在Admin里是否可编辑
13、error_messages=None　　错误提示
14、auto_created=False　　自动创建
15、help_text　　在Admin中提示帮助信息
16、validators=[]  验证
17、upload-to  文件上传路径
```

## 关系
### 多对一
使用ForeignKey定义

这里Tag是Pic的外键，一个Pic对应一个Tag，一个Tag对应多个Pic
每个图片只能有一个标签，每个标签可以属于多个图片
文件：**myDjango/myDjango/toon/models.py**
```
class Tag(models.Model):
    tagname = models.CharField(max_length=32)
    def __unicode__(self):
        return self.tagname

class Pic(models.Model):
    picname = models.CharField(max_length=32)
    pic = models.ImageField(null=True,blank=True)
    tags = models.ForeignKey(Tag)
    def __unicode__(self):
        return self.picname
```
### 多对多
使用ManyToManyField定义

把上例改为多对多关系，一个Pic对应多个Tag，一个Tag对应多个Pic
每个图片可以有多个标签，每个标签可以属于多个图片
```
class Tag(models.Model):
    tagname = models.CharField(max_length=32)
    def __unicode__(self):
        return self.tagname

class Pic(models.Model):
    picname = models.CharField(max_length=32)
    pic = models.ImageField(null=True,blank=True,upload_to='uploads')
    tags = models.ManyToManyField(Tag) 
    def __unicode__(self):
        return self.picname
```

### 一对一
使用OneToOneField定义

# Meta
使用(Meta)元数据设置默认排序方式
```
class Tag(models.Model):
    #...
    class Meta:
        ordering = ['tagname']

class Pic(models.Model):
    #...
    class Meta:
        ordering = ['picname']
```


# 更新数据库
```
python manage.py makemigrations //生成py文件
python manage.py migrate
```
启动数据库交互
```
python manage.py dbshell
```

基本操作
```
# 增--3种方法
# 第1种方法
models.tb.objects.create(c1='xx', c2='oo') 
# 第2种方法
obj = models.tb(c1='xx', c2='oo')
obj.save()
# 第3种方法--可以接受字典类型数据 **kwargs
dic = {'c1':'xx','c2':'oo'}
models.tb.objects.create(**dic)

# 查
models.tb.objects.get(id=123)         #获取单条数据，不存在则报错（不建议）
models.tb.objects.all()               #获取全部数据
models.tb.objects.all().first()       #获取全部数据的第1条数据
models.tb.objects.filter(name='seven') #1个参数，获取指定条件的数据
models.tb.objects.filter(name='seven',password='123') #2个参数，获取指定条件的数据
dic = {'name':'seven','password':'123'}  
models.tb.objects.filter(**dic)       #参数可以是字典形式，获取指定条件的数据

# 删
models.tb.objects.filter(name='seven').delete() # 删除指定条件的数据
dic = {'name':'seven','password':'123'}
models.tb.objects.filter(**dic).delete()  #参数是字典形式

# 改
# 第1种方法
models.tb.objects.filter(name='seven').update(password='123') #将指定条件的数据更新
# 第2种方法
obj = models.tb.objects.get(id=1)
obj.c1 = '111'
obj.save()        # 修改单条数据
# 第3种方法--字典参数
dic = {'name':'seven','password':'123'}
models.tb.objects.filter(**dic).update(**dic)  #参数是字典形式
```

```
>>> from toon.models import Pic
>>> p1 = Pic(picname="aaa",pic="",tips="a")
>>> p1.save()
>>> ps = Pic.objects.all()
>>> for p in ps:
...  print p.picname
... 
aaa
>>> ps = Pic.objects.all().order_by("picname")
>>> for p in ps:
...  print p.picname
... 
aaa
bbb
>>> ps = Pic.objects.all().order_by("-picname")
>>> for p in ps:
...  print p.picname
... 
bbb
aaa
```

# 应用举例
## 编写路由
在主文件中添加路由
文件：**myDjango/myDjango/urls.py**
```
from django.conf.urls import patterns, url
from myDjango.views import hello, current_datetime, hours_ahead
from toon.views import showpic

urlpatterns = [
    #...
    url(r'^toon/pic$',showpic)
]
```

## 编写视图函数
在应用文件中添加视图函数，注意要导入模型文件
文件：**myDjango/myDjango/toon/views.py**
```
from django.http import HttpResponse

from toon.models import Pic

def showpic(request):
    ps_str = ''
    ps = Pic.objects.all()
    for p in ps:
        ps_str += p.picname
        ps_str += ' '
    return HttpResponse(ps_str)
```
## 分开路由
对于不同功能的应用，可以把路由放在不同文件中

在主文件夹的路由中使用include
文件：**myDjango/myDjango/urls.py**
```
from django.conf.urls import patterns, url, include

from myDjango.views import hello, current_datetime, hours_ahead
from toon.views import showpic
from django.contrib import admin

urlpatterns = [
    #...
    url(r'^toon/',include('toon.urls'))
]
```
文件：**myDjango/myDjango/toon/urls.py**
```
from django.conf.urls import patterns, url, include

from toon.views import showpic
from django.contrib import admin

urlpatterns = [
    url(r'^pic/',showpic)
]
```
# admin
## 配置
在配置文件中已经默认添加了admin功能
```
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'toon',
]

MIDDLEWARE_CLASSES = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.auth.middleware.SessionAuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]
```

## 添加管理路由
admin界面默认位于[site]/admin这个URL
文件：**myDjango/myDjango/urls.py**
```
from django.conf.urls import patterns, url, include

from myDjango.views import hello, current_datetime, hours_ahead
from toon.views import showpic
from django.contrib import admin

urlpatterns = [
    url(r'^admin/', include(admin.site.urls)),
    #...
]
```
为了让admin界面能够管理某个数据模型，修改admin.py文件
文件：**myDjango/myDjango/toon/admin.py**
```
from django.contrib import admin

from toon.models import Pic

admin.site.register(Pic)
```

## 创建超级用户
在shell界面创建超级用户
```
python manage.py createsuperuser
```

## 自定义页面
### 添加搜索
文件：**myDjango/myDjango/toon/admin.py**
```
class PicAdmin(admin.ModelAdmin):
    search_fields = ('picname', 'tips')
    #...
```

### 自定义显示列表
默认只显示对象的__unicode__值
```
class PicAdmin(admin.ModelAdmin):
    list_display = ('picname','tips')
    #...
```

### 过滤器
根据字段筛选过滤
```
class PicAdmin(admin.ModelAdmin):
    list_filter = ('tips',)
    #...
```

### 排序
如果是降序，就在前面加个'-'号
```
class TagAdmin(admin.ModelAdmin):
    ordering = ('tagname',)
```

## 自定义编辑页面
当使用admin管理数据库时，可以自定义编辑页面
只显示指定的内容
文件：**myDjango/myDjango/toon/admin.py**
```
from django.contrib import admin

from toon.models import Pic, Tag

class TagAdmin(admin.ModelAdmin):
    fields = ('pic', 'tags')

class PicAdmin(admin.ModelAdmin):
    fields = ('picname', 'pic')

admin.site.register(Tag, TagAdmin)
admin.site.register(Pic, PicAdmin)
```

也可以把不同内容分开显示分割成字段集
也可以给每个字段添加类
文件：**myDjango/myDjango/toon/admin.py**
```
class PicAdmin(admin.ModelAdmin):
    fieldsets = (
        [
            'Main',{
                'fields': ('picname', 'pic'),
            }
        ],
        [
            'Advance',{
                'classes': ('collapse',), #CSS 隐藏
                'fields': ('tips',),
            }
        ]
    )
```

### inline
在Pic页面，通过inline把Tag也显示出来
文件：**myDjango/myDjango/toon/admin.py**
```
class TagInLine(admin.TabularInline):
    model = Tag

class PicAdmin(admin.ModelAdmin):
    inlines = [TagInLine]
    #...
```

### filter_horizontal
只能用于多对多关系
在创建一个Pic对象时，在选择Tag的页面可以使用搜索功能，方便查找
```
class PicAdmin(admin.ModelAdmin):
    filter_horizontal = ('tags',)
```

[参考](http://www.cnblogs.com/maociping/p/5255834.html)
