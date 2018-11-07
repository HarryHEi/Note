---
title: myblog-04 用户注册
date: 2016-04-14 10:38:49
tags: [flask]
---

# 电子邮件
[邮件配置参数](http://harryhei.github.io/2016/03/13/flask-send-mail/)
## 配置
QQ邮箱的发送邮件服务器和端口：**smtp.qq.com ，使用SSL，端口号465**
密码要从环境中导入，避免直接写入脚本
```
import os

app.config['MAIL_SERVER'] = 'smtp.qq.com'
app.config['MAIL_PORT'] = 587
app.config['MAIL_USE_TLS'] = True
app.config['MAIL_USERNAME'] = os.environ.get('MAIL_USERNAME')
app.config['MAIL_PASSWORD'] = os.environ.get('MAIL_PASSWORD')
```
## 初始化
```
from flask.ext.mail import Mail
mail = Mail(app)
```
### 集成发送邮件功能
异步发送邮件
文件：app/email.py
```
from threading import Thread
from flask import current_app, render_template
from flask.ext.mail import Message

app.config['FLASKY_MAIL_SUBJECT_PREFIX'] = '[Harry]'
app.config['FLASKY_MAIL_SENDER'] = 'harryx520@qq.com'

def send_async_email(app, msg):
    with app.app_context():
        mail.send(msg)

def send_email(to, subject, template, **kwargs):
    app = current_app._get_current_object()
    msg = Message(app.config['FLASKY_MAIL_SUBJECT_PREFIX'] + ' ' + subject,
                  sender=app.config['FLASKY_MAIL_SENDER'], recipients=[to])
    msg.body = render_template(template + '.txt', **kwargs)
    msg.html = render_template(template + '.html', **kwargs)
    thr = Thread(target=send_async_email, args=[app, msg])
    thr.start()
    return thr
```
# 注册用户
## 建立表单
文件：app/auth/forms.py
Regexp提供验证函数，确保username字段合法
EqualTo实验密码的第二次输入核对
```
from flask.ext.wtf import Form
from wtforms import StringField, PasswordField, BooleanField, SubmitField
from wtforms.validators import Required, Length, Email, Regexp, EqualTo
from wtforms import ValidationError

from ..models import User

class RegistrationForm(Form):
    email = StringField('Email', validators=[Required(), Length(1, 64),
                                           Email()])
    name = StringField('name', validators=[
        Required(), Length(1, 64), Regexp('^[A-Za-z][A-Za-z0-9_.]*$', 0,
                                          'Names must have only letters, '
                                          'numbers, dots or underscores')])
    password = PasswordField('Password', validators=[
        Required(), EqualTo('password2', message='Passwords must match.')])
    password2 = PasswordField('Confirm password', validators=[Required()])
    submit = SubmitField('Register')

    def validate_email(self, field):
        if User.query.filter_by(email=field.data).first():
            raise ValidationError('Email already registered.')

    def validate_name(self, field):
        if User.query.filter_by(name=field.data).first():
            raise ValidationError('Name already in use.')
```
注册页面：**app/templates/auth/register.html**
```
{% extends "base.html" %}
{% import "bootstrap/wtf.html" as wtf %}

{% block title %}Flasky - Register{% endblock %}

{% block page_content %}
<div class="page-header">
    <h1>Register</h1>
</div>
<div class="col-md-4">
    {{ wtf.quick_form(form) }}
</div>
{% endblock %}
```
登录页面添加到注册页面的链接
文件：**app/templates/auth/login.html**
```
<p>New user? <a href="{{ url_for('auth.register') }}">Click here to register</a>.</p>
```
## 编写视图函数
文件：app/auth/views.py
```
from flask import render_template, redirect, request, url_for, flash
from flask.ext.login import login_user, logout_user, login_required,current_user

from . import auth
from .. import db
from ..models import User
from ..email import send_email
from .forms import LoginForm,RegistrationForm

@auth.route('/register', methods=['GET', 'POST'])
def register():
    form = RegistrationForm()
    if form.validate_on_submit():
        user = User(email=form.email.data,
                    name=form.name.data,
                    password=form.password.data)
        db.session.add(user)
        flash('You can now login.')
        return redirect(url_for('auth.login'))
    return render_template('auth/register.html', form=form)
```
## 确认账户
证明用户注册时提供的邮箱是本人邮箱
### 使用itsdangerous生成确认令牌
确认邮件发送http://example.com/auth/config/id这种形式的URL，其中id是数据库分配给用户的id
安全的做法是将URL中的id换成相同信息安全加密后得到的令牌
```
>>> from manage import app
>>> from itsdangerous import TimedJSONWebSignatureSerializer as Serializer

#生成具有过期时间的JSON Web签名
>>> s = Serializer(app.config['SECRET_KEY'], expires_in = 3600)

#为指定数据生成一个加密签名
>>> token = s.dumps({ 'confirm': 23 })
>>> token
'eyJhbGciOiJIUzI1NiIsImV4(...)30.UxML4AVrfSx6Mz0-hLzp5hy6Qayoht5J78IDr_-NNug'

#解码令牌，这个指令会检验签名和过期时间，如果通过，返回原始数据
>>> data = s.loads(token)
>>> data
{u'confirm': 23}
```
generate_confirmation_token()生成一个令牌，有效期为1小时
confirm()检验令牌与用户是否匹配
更新用户模型后要进行一次数据库更新
### 在用户模型中添加确认令牌的功能
文件：app/models.py
```
from itsdangerous import TimedJSONWebSignatureSerializer as Serializer
from flask import current_app

class User(UserMixin, db.Model):

    #...
    confirmed = db.Column(db.Boolean, default=False)

    def generate_confirmation_token(self, expiration=3600):
        s = Serializer(current_app.config['SECRET_KEY'], expiration)
        return s.dumps({'confirm': self.id})

    def confirm(self, token):
        s = Serializer(current_app.config['SECRET_KEY'])
        try:
            data = s.loads(token)
        except:
            return False
        if data.get('confirm') != self.id:
            return False
        self.confirmed = True
        db.session.add(self)
        return True
```
### 发送确认邮件
当/register路由把用户添加到数据库时，会重定向到/index，在重定向之前，这个路由需要发送确认邮件
文件：**app/auth/views.py**
```
@auth.route('/register', methods=['GET', 'POST'])
def register():
    form = RegistrationForm()
    if form.validate_on_submit():
        user = User(email=form.email.data,
                    name=form.name.data,
                    password=form.password.data)
        db.session.add(user)
        db.session.commit()
        token = user.generate_confirmation_token()
        send_email(user.email, 'Confirm Your Account',
                   'auth/email/confirm', user=user, token=token)
        flash('Please confirm the email!')
        return redirect(url_for('auth.login'))
    return render_template('auth/register.html', form=form)
```
### 添加邮件文本
使用_external=True参数，指定使用绝对URL
文件：**app/templates/auth/email/condirm.txt**
```
Dear {{ user.name }},

Welcome to Harry's blog!

To confirm your account please click on the following link:

{{ url_for('auth.confirm', token=token, _external=True) }}

Sincerely
```
### 确认用户视图函数
Flask-Login提供的login_required修饰器会保护这个路由
用户点击邮件中的链接后要先登录，才能执行这个视图函数
文件：**app/auth/views.py**
```
from flask.ext.login import current_user

@auth.route('/confirm/<token>')
@login_required
def confirm(token):
    if current_user.confirmed:
        return redirect(url_for('main.index'))
    if current_user.confirm(token):
        flash('You have confirmed your account. Thanks!')
    else:
        flash('The confirmation link is invalid or has expired.')
    return redirect(url_for('main.index'))
```
### 处理没有确认的账户
当满足以下三个条件时，before_app_request处理程序会拦截请求

1 . 用户已登录(**current_user.is_authenticated()** 必须返回 True)
2 . 用户账户没有确认
3 . 请求的端点不在认证蓝本中，访问认证路由要获取权限

文件：**app/auth/views.py**
```
@auth.before_app_request
def before_request():
    if current_user.is_authenticated \
            and not current_user.confirmed \
            and request.endpoint[:5] != 'auth.' \
            and request.endpoint != 'static':
        return redirect(url_for('auth.unconfirmed'))

@auth.route('/unconfirmed')
def unconfirmed():
    if current_user.is_anonymous or current_user.confirmed:
        return redirect(url_for('main.index'))
    return render_template('auth/unconfirmed.html')
```
### 重新发送账户确认邮件
显示给未确认用户的页面有一个再次发送确认邮件的链接
文件：**app/auth/views.py**
```
@auth.route('/confirm')
@login_required
def resend_confirmation():
    token = current_user.generate_confirmation_token()
    send_email(current_user.email, 'Confirm Your Account',
               'auth/email/confirm', user=current_user, token=token)
    flash('A new confirmation email has been sent to you by email.')
    return redirect(url_for('main.index'))
```
