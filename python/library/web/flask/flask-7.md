---
title: myblog-07 用户资料
date: 2016-04-17 13:18:51
tags: [flask]
---


# 用户资料
## 更新数据库
### 编写用户模板
在用户模板中添加所在地、个性签名、注册日期和最后访问日期
数据库以datetime格式存储UTC时间，Flask-Moment自动更具时区渲染时间
首先引入moment.js库
文件：**app/templates/base.html**
```
{% block scripts %}
{{ super() }}
{{ moment.include_moment() }}
{{ moment.lang("zh-CN") }} 
{% endblock %}
```
渲染时间
文件：**app/templates/user.html**
```
<p>Member since {{ moment(user.member_since).format('LL') }}.</p>
<p>Last seen {{ moment(user.last_seen).fromNow() }}.</p>
```
db.String()和db.Text()的区别是后者不需要指定长度
这里的member_since和last_seen分别是注册日期和最后访问日期，默认值都是now函数的值
如果数据库提交时，没有赋值，则提交默认值，而不是NULL
文件：**app/models.py**
```
from datetime import datetime

class User(UserMixin, db.Model):

    #...
    location = db.Column(db.String(64))
    about_me = db.Column(db.Text())
    member_since = db.Column(db.DateTime(), default=datetime.now)
    last_seen = db.Column(db.DateTime(), default=datetime.now)
```
### 刷新最后访问日期
添加用户刷新时间功能
文件：**app/models.py**
```
class User(UserMixin, db.Model):

    #...
    def ping(self):
        self.last_seen = datetime.now()
        db.session.add(self)
```
auth蓝本中的before_app_request处理程序会在用户每次请求之前运行
文件：**app/auth/views.py**
```
@auth.before_app_request
def before_request():
    if current_user.is_authenticated:
        current_user.ping()
        #...
```
## 用户资料页面
### 编写路由
从URL接收参数name，如果用户不存在，返回404错误
文件：**app/main/views.py**
```
@main.route('/user/<name>')
def user(name):
    user = User.query.filter_by(name=name).first_or_404()
    return render_template('user.html', user=user)
```
### 资料页面模板
只有管理员可以看到邮件地址
文件：**app/templates/user.html**
```
{% extends "base.html" %}

{% block title %}{{ user.name }}{% endblock %}

{% block page_content %}
<div class="page-header">
    <h1>{{ user.name }}</h1>
    {% if user.name or user.location %}
    <p>
        {% if user.location %}
            From <a href="http://maps.google.com/?q={{ user.location }}">{{ user.location }}</a>
        {% endif %}
    </p>
    {% endif %}
    {% if current_user.is_administrator() %}
    <p>{{ user.email }}</a></p>
    {% endif %}
    {% if user.about_me %}<p>{{ user.about_me }}</p>{% endif %}
    <p>Member since {{ moment(user.member_since).format('LL') }}.</p>
    <p>Last seen {{ moment(user.last_seen).fromNow() }}.</p>
</div>
{% endblock %}
```
### 查看个人资料
在导航条创建一个链接，用于访问自己的资料
文件：**app/templates/base.html**
```
{% if current_user.is_authenticated %}
<li class="dropdown">
    <a href="#" class="dropdown-toggle" data-toggle="dropdown">Account <b class="caret"></b></a>
    <ul class="dropdown-menu">
        <li><a href="{{ url_for('main.user', name=current_user.name) }}">Profile</a></li>
        #...
    </ul>
</li>
{% else %}
```
## 资料编辑器
用户可以通过资料编辑器更改自己的资料
管理员可以通过资料编辑器更改所有用户资料和角色
针对两个角色创建不同的表单
### 用户级别的资料编辑器
这个表单中的所有字段都是可选的，可以不填
文件：**app/main/forms.py**
```
class EditProfileForm(Form):
    location = StringField('Location', validators=[Length(0, 64)])
    about_me = TextAreaField('About me')
    submit = SubmitField('Submit')
```
渲染表单
文件：**app/templates/edit_profile.html** 
```
{% extends "base.html" %}
{% import "bootstrap/wtf.html" as wtf %}

{% block title %}Edit Profile{% endblock %}

{% block page_content %}
<div class="page-header">
    <h1>Edit Your Profile</h1>
</div>
<div class="col-md-4">
    {{ wtf.quick_form(form) }}
</div>
{% endblock %}
```
编辑路由
在显示表单之前，存下初始值，如果没有提交更改，返回默认值
文件：**app/main/views.py**
```
@main.route('/edit-profile', methods=['GET', 'POST'])
@login_required
def edit_profile():
    form = EditProfileForm()
    if form.validate_on_submit():
        current_user.location = form.location.data
        current_user.about_me = form.about_me.data
        db.session.add(current_user)
        flash('Your profile has been updated.')
        return redirect(url_for('.user', name=current_user.name))
    form.location.data = current_user.location
    form.about_me.data = current_user.about_me
    return render_template('edit_profile.html', form=form)
```
添加到编辑页面的链接
文件：**app/templates/user.html**
```
<p>
    {% if user == current_user %}
        <a class="btn btn-default" href="{{ url_for('.edit_profile') }}">Edit Profile</a>
    {% endif %}
</p>
```
关于数据库中文乱码的问题，创建表的时候，指定编码格式：**default charset=utf8**

### 管理员级别的资料编辑器
管理员使用的表单比普通用户复杂，除了地点、个性签名还有用户名、电子邮件、确认状态和角色
SelectField实例必须在choice属性中设置各选项，选项是一个元组组成的列表
这里使用一个查询按照角色名的字母排序的所有角色
SelectField构造函数中添加coerce=int参数，从而把字段值转化为整数，而不使用字符串
email和name字段的构造方式和认证表单中一样
文件：**app/main/forms.py**
```
class EditProfileAdminForm(Form):
    email = StringField('Email', validators=[Required(), Length(1, 64),
                                             Email()])
    confirmed = BooleanField('Confirmed')
    name = StringField('Name', validators=[
        Required(), Length(1, 64), Regexp('^[A-Za-z][A-Za-z0-9_.]*$', 0,
                                          'Names must have only letters, '
                                          'numbers, dots or underscores')])
    role = SelectField('Role', coerce=int)
    location = StringField('Location', validators=[Length(0, 64)])
    about_me = TextAreaField('About me')
    submit = SubmitField('Submit')

    def __init__(self, user, *args, **kwargs):
        super(EditProfileAdminForm, self).__init__(*args, **kwargs)
        self.role.choices = [(role.id, role.name)
                             for role in Role.query.order_by(Role.name).all()]
        self.user = user

    def validate_email(self, field):
        if field.data != self.user.email and \
                User.query.filter_by(email=field.data).first():
            raise ValidationError('Email already registered.')

    def validate_name(self, field):
        if field.data != self.user.name and \
                User.query.filter_by(name=field.data).first():
            raise ValidationError('Name already in use.')
```
视图函数，通过URL获得用户id，如果用户不存在，返回404
user.role是通过Role查询赋值
文件：**app/main/views.py**
```
@main.route('/edit-profile/<int:id>', methods=['GET', 'POST'])
@login_required
@admin_required
def edit_profile_admin(id):
    user = User.query.get_or_404(id)
    form = EditProfileAdminForm(user=user)
    if form.validate_on_submit():
        user.email = form.email.data
        user.name = form.name.data
        user.confirmed = form.confirmed.data
        user.role = Role.query.get(form.role.data)
        user.location = form.location.data
        user.about_me = form.about_me.data
        db.session.add(user)
        flash('The profile has been updated.')
        return redirect(url_for('.user', name=user.name))
    form.email.data = user.email
    form.confirmed.data = user.confirmed
    form.role.data = user.role_id
    form.name.data = user.name
    form.location.data = user.location
    form.about_me.data = user.about_me
    return render_template('edit_profile.html', form=form, user=user)
```
在用户的资料页面添加管理员管理链接
文件：**app/templates/user.html**
```
{% if current_user.is_administrator() %}
    <a class="btn btn-danger" href="{{ url_for('.edit_profile_admin', id=user.id) }}">Edit Profile [Admin]</a>
{% endif %}
```
## 用户头像
这里使用的是将图片的URL储存在数据库，需要更新用户模型
文件：**app/models.py**
```
class User(UserMixin, db.Model):  
    #...
    avatar = db.Column(db.String(128))
    #...
```
视图函数也稍微修改
文件：**app/main/views.py**
```
@main.route('/edit-profile', methods=['GET', 'POST'])
@login_required
def edit_profile():
    if form.validate_on_submit():
        current_user.avatar = form.avatar.data
        #...
        return redirect(url_for('.user', name=current_user.name))
    form.avatar.data = current_user.avatar
    #...

@main.route('/edit-profile/<int:id>', methods=['GET', 'POST'])
@login_required
@admin_required
def edit_profile_admin(id):
    user = User.query.get_or_404(id)
    form = EditProfileAdminForm(user=user)
    if form.validate_on_submit():
        user.avatar = form.avatar.data
        #...
        return redirect(url_for('.user', name=user.name))
    form.avatar.data = user.avatar
    #...
```
然后在用户资料页面渲染
文件：**app/templates/user.html**
```
{% if user.avatar %}
    <img class="img-rounded profile-thumbnail" src="{{ user.avatar }}" width="200px" height="200px">
{% endif %}
```
