---
title: myblog-0e 部署
date: 2016-04-24 14:49:56
tags: [flask]
---

# 部署
## 部署流程
文件： `manage.py`
```
@manager.command
def deploy():
    """Run deployment tasks."""
    from flask.ext.migrate import upgrade
    from app.models import Role, User

    # 数据库迁移更新
    upgrade()

    # 创建角色
    Role.insert_roles()

    # 自我关注
    User.add_self_follows()
```
运行deploy更新
```
(venv)$ python manage.py deploy
```
## 把生产环境中的错误写入日志
出错时发送电子邮件
文件： `config.py`
```
class ProductionConfig(Config):
    SQLALCHEMY_DATABASE_URI ='mysql://flask:admin@localhost/flaskdb'

    @classmethod
    def init_app(cls, app):
        Config.init_app(app)

        # email errors to the administrators
        import logging
        from logging.handlers import SMTPHandler
        credentials = None
        secure = None
        if getattr(cls, 'MAIL_USERNAME', None) is not None:
            credentials = (cls.MAIL_USERNAME, cls.MAIL_PASSWORD)
            if getattr(cls, 'MAIL_USE_TLS', None):
                secure = ()
        mail_handler = SMTPHandler(
            mailhost=(cls.MAIL_SERVER, cls.MAIL_PORT),
            fromaddr=cls.FLASKY_MAIL_SENDER,
            toaddrs=[cls.FLASKY_ADMIN],
            subject=cls.FLASKY_MAIL_SUBJECT_PREFIX + ' Application Error',
            credentials=credentials,
            secure=secure)
        mail_handler.setLevel(logging.ERROR)
        app.logger.addHandler(mail_handler)
```
## Gunicorn
Gunicorn是一个给Unix用的WSGI HTTP服务器
运行一个Flask应用
```
(venv)$ gunicorn manage:app
```
用4个worker进程运行一个flask应用，绑定到localhost的4000端口
```
(venv)$ gunicorn -w 4 -b 127.0.0.1:4000 manage:app
```
## 使用Flask-SSLify启用安全HTTP
文件：app/__init__.py
```
def create_app(config_name):

    #...
    if not app.debug and not app.testing and not app.config['SSL_DISABLE']:
        from flask.ext.sslify import SSLify
        sslify = SSLify(app)
```
# 后台运行
后台运行，记录日志
```
python manage.py runserver >> flask.log 2>&1 &
python manage.py runserver &>>flask.log 
```

# 语言问题
```
apt-get install localepurge //选择en_US.UTF-8 和 zh_CN.UTF-8
locale-gen zh_CN.UTF-8 en_US.UTF-8 //生成语言
locale //打印配置
```
# 环境
```
apt-get install python-virtualenv
apt-get install python-pip
source venv/bin/activate
pip install -r requirements/common.txt
```
# mysql
```
apt-get install mysql-server
apt-get install python-setuptools
apt-get install libmysqld-dev
apt-get install libmysqlclient-dev
apt-get install python-dev
easy_install mysql-python
```
创建新用户
```
mysql -u root -p //登录
use mysql
create user "flask"@"localhost" identified by "admin"
flush privileges  //刷新
create database flaskdb //创建数据库
grant all privileges on flaskdb.* to flask@localhost identified  by 'admin' //赋予权限
flush privileges 
```

# 终语
至此，当初搭建博客的目标基本达成
后面是更基础的学习
