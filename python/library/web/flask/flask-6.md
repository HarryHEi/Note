---
title: myblog-06 角色和权限
date: 2016-04-16 10:36:56
tags: [flask]
---

# 角色和权限
## 角色在数据库中的表示
常见的角色为用户和管理员，他们拥有不同的权限
在数据库模型中添加一个角色模型，针对不同角色给与不同权限
flag标识为True时，表示该角色为默认角色，只能有一个角色为True
Role通过relationship()与User关联

### 用户角色数据库模型
文件：**app/models.py**
```
class Role(db.Model):
    __tablename__="roles"
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(64), unique=True, index=True)
    flag = db.Column(db.Boolean, default=False, index=True)
    permissions = db.Column(db.Integer)
    users = db.relationship('User', backref='role', lazy='dynamic')

#...

class User(UserMixin, db.Model):
    __tablename__ = 'users'
    id = db.Column(db.Integer, primary_key=True)
    email = db.Column(db.String(64), unique=True, index=True)
    name = db.Column(db.String(64), unique=True, index=True)
    password_hash = db.Column(db.String(128))
    role_id = db.Column(db.Integer, db.ForeignKey('roles.id'))
    confirmed = db.Column(db.Boolean, default=False)
```
### mysql数据库表属性
```
mysql> desc users;
+---------------+------------+------+-----+---------+----------------+
| Field         | Type       | Null | Key | Default | Extra          |
+---------------+------------+------+-----+---------+----------------+
| id            | int(32)    | NO   | PRI | NULL    | auto_increment |
| email         | char(64)   | YES  | UNI | NULL    |                |
| name          | char(64)   | YES  | UNI | NULL    |                |
| password_hash | char(128)  | YES  |     | NULL    |                |
| confirmed     | tinyint(1) | YES  |     | NULL    |                |
| role_id       | int(32)    | YES  |     | NULL    |                |
+---------------+------------+------+-----+---------+----------------+

mysql> desc roles;
+-------------+------------+------+-----+---------+----------------+
| Field       | Type       | Null | Key | Default | Extra          |
+-------------+------------+------+-----+---------+----------------+
| id          | int(32)    | NO   | PRI | NULL    | auto_increment |
| name        | char(64)   | YES  | UNI | NULL    |                |
| permissions | int(32)    | YES  |     | NULL    |                |
| flag        | tinyint(1) | YES  |     | NULL    |                |
+-------------+------------+------+-----+---------+----------------+
```

## 权限设置
### 程序的权限

操作 | 位值 | 说明
------|------|------
关注用户 | 0x01 | 关注其他用户
评论 | 0x02 | 在他人的文章中发表评论
写文章 | 0x04 |自己写原创文章
管理他人评论 | 0x08 | 查处他人发表的不当评论
管理员 | 0x80 | 管理网站

### 代码表示权限
文件：**app/models.py**
```
class Permission:
    FOLLOW = 0x01
    COMMENT = 0x02
    WRITE_ARTICLES = 0x04
    MODERATE_COMMENTS = 0x08
    ADMINISTER = 0x80
```

### 拥有不同权限的角色

角色 | 权限 | 说明
-------|--------|------
匿名 | 0x00 | 没有登录的用户，只有阅读权限
用户 | 0x07 | 正常用户，拥有管理以外的权限
协管员 | 0x0f | 具有审查评论的权限
管理员 | 0xff | 拥有所有权限

## 在数据库中创建角色
自动添加角色
roles字典内容有两个参数，一个参数是对应的权限相或(加)，另一个参数是flag，表示是否为默认角色
文件：**app/models.py**
```
class Role(db.Model):
    
    #...
    @staticmethod
    def insert_roles():
        roles = {
            'User': (Permission.FOLLOW |
                     Permission.COMMENT |
                     Permission.WRITE_ARTICLES, True),
            'Moderator': (Permission.FOLLOW |
                          Permission.COMMENT |
                          Permission.WRITE_ARTICLES |
                          Permission.MODERATE_COMMENTS, False),
            'Administrator': (0xff, False)
        }
        for r in roles:
            role = Role.query.filter_by(name=r).first()
            if role is None:
                role = Role(name=r)
            role.permissions = roles[r][0]
            role.flag = roles[r][1]
            db.session.add(role)
        db.session.commit()
```
通过shell会话把角色加入数据库
```
(venv) $ python manage.py shell
>>> Role.insert_roles()
>>> Role.query.all()
[<Role u'Administrator'>, <Role u'User'>, <Role u'Moderator'>]
```

## 赋予角色
在用户注册时赋予用户角色，通过环境变量读取**FLASKY_ADMIN**获得管理员电子邮件
如果注册的是管理员，则拥有管理员权限，否则为设置的默认角色
文件：**app/models.py**
```
class User(UserMixin, db.Model):

    #...
    def __init__(self, **kwargs):
        super(User, self).__init__(**kwargs)
        if self.role is None:
            if self.email == current_app.config['FLASKY_ADMIN']:
                self.role = Role.query.filter_by(permissions=0xff).first()
            if self.role is None:
                self.role = Role.query.filter_by(flag=True).first()
```

## 角色验证
### 检查角色权限的函数
AnonymousUser类表示未登录的用户，所以不用检查用户是否登录，可以直接调用**current_user.can()**
文件：**app/models.py**
```
from flask.ext.login import UserMixin, AnonymousUserMixin

class User(UserMixin, db.Model):

    #...
    def can(self, permissions):
        return self.role is not None and \
            (self.role.permissions & permissions) == permissions

    def is_administrator(self):
        return self.can(Permission.ADMINISTER)

class AnonymousUser(AnonymousUserMixin):
    def can(self, permissions):
        return False

    def is_administrator(self):
        return False
```

### 检查用户权限的修饰器
两个修饰器都使用Python标准库的functools包
如果用户不具有指定权限，则返回403错误码
需要添加403.html文件和相应路由
文件：**app/decorators.py**
```
from functools import wraps
from flask import abort
from flask.ext.login import current_user

from .models import Permission


def permission_required(permission):
    def decorator(f):
        @wraps(f)
        def decorated_function(*args, **kwargs):
            if not current_user.can(permission):
                abort(403)
            return f(*args, **kwargs)
        return decorated_function
    return decorator


def admin_required(f):
    return permission_required(Permission.ADMINISTER)(f)
```
用法如下
```
from decorators import admin_required, permission_required
from .models import Permission

@main.route('/admin')
@login_required
@admin_required
def for_admins_only():
    return "For administrators!"

@main.route('/moderator')
@login_required
@permission_required(Permission.MODERATE_COMMENTS)
def for_moderators_only():
    return "For comment moderators!"
```
Permission类加入模板上下文
文件：**app/main/__init__.py**
```
from ..models import Permission

@main.app_context_processor
def inject_permissions():
    return dict(Permission=Permission)
```
