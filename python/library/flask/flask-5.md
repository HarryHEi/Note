---
title: myblog-05 管理账户
date: 2016-04-15 19:53:37
tags: [flask]
---

# 管理账户
## 修改密码
在登录的前提下，提供修改密码的功能
### 添加表单
文件：**app/auth/forms.py**
```
class ChangePasswordForm(Form):
    old_password = PasswordField('Old password', validators=[Required()])
    password = PasswordField('New password', validators=[
        Required(), EqualTo('password2', message='Passwords must match')])
    password2 = PasswordField('Confirm new password', validators=[Required()])
    submit = SubmitField('Update Password')
```
### 视图函数
文件：**app/auth/views.py**
```
from .forms import ChangePasswordForm

@auth.route('/change-password', methods=['GET', 'POST'])
@login_required
def change_password():
    form = ChangePasswordForm()
    if form.validate_on_submit():
        if current_user.verify_password(form.old_password.data):
            current_user.password = form.password.data
            db.session.add(current_user)
            flash('Your password has been updated.')
            return redirect(url_for('main.index'))
        else:
            flash('Invalid password.')
    return render_template("auth/change_password.html", form=form)
```
### 编写页面
文件：**app/templates/auth/change_password.html**
```
{% extends "base.html" %}
{% import "bootstrap/wtf.html" as wtf %}

{% block title %}Change Password{% endblock %}

{% block page_content %}
<div class="page-header">
    <h1>Change Your Password</h1>
</div>
<div class="col-md-4">
    {{ wtf.quick_form(form) }}
</div>
{% endblock %}
```
### 添加改变密码的标签
提供一个菜单，可以选择登出或者更改密码
文件 **app/templates/base.html**
```
{% if current_user.is_authenticated %}
<li class="dropdown">
    <a href="#" class="dropdown-toggle" data-toggle="dropdown">Account <b class="caret"></b></a>
    <ul class="dropdown-menu">
        <li><a href="{{ url_for('auth.change_password') }}">Change Password</a></li>
        <li><a href="{{ url_for('auth.logout') }}">Log Out</a></li>
    </ul>
</li>
{% else %}
<li><a href="{{ url_for('auth.login') }}">Log In</a></li>
{% endif %}
```
## 忘记密码
使用邮件令牌提供给用户修改密码
### 添加表单
提供两个表单，一个用于输入需要重置密码的邮件地址，另一个用于通过邮件确认后给与用户重置密码
文件：**app/auth/forms.py**
```
class PasswordResetRequestForm(Form):
    email = StringField('Email', validators=[Required(), Length(1, 64),
                                             Email()])
    submit = SubmitField('Reset Password')


class PasswordResetForm(Form):
    email = StringField('Email', validators=[Required(), Length(1, 64),
                                             Email()])
    password = PasswordField('New Password', validators=[
        Required(), EqualTo('password2', message='Passwords must match')])
    password2 = PasswordField('Confirm password', validators=[Required()])
    submit = SubmitField('Reset Password')

    def validate_email(self, field):
        if User.query.filter_by(email=field.data).first() is None:
            raise ValidationError('Unknown email address.')
```
### 编写页面
在登录界面提供重置密码的链接
文件：**app/templates/auth/login.html**
```
<p>Forgot your password? 
    <a href="{{ url_for('auth.password_reset_request') }}">Click here to reset it</a>.
</p>
```
显示给用户，用来输入邮件，或者重置密码
文件：**app/templates/auth/reset_password.html**
```
{% extends "base.html" %}
{% import "bootstrap/wtf.html" as wtf %}

{% block title %}Password Reset{% endblock %}

{% block page_content %}
<div class="page-header">
    <h1>Reset Your Password</h1>
</div>
<div class="col-md-4">
    {{ wtf.quick_form(form) }}
</div>
{% endblock %}
```
### 视图函数
视图函数提供两个路由，一个提供重置密码的链接，另一个是邮件发送给用户的链接
文件：**app/auth/views.py**
```
from .forms import PasswordResetRequestForm, PasswordResetForm

@auth.route('/reset', methods=['GET', 'POST'])
def password_reset_request():
    if not current_user.is_anonymous:
        return redirect(url_for('main.index'))
    form = PasswordResetRequestForm()
    if form.validate_on_submit():
        user = User.query.filter_by(email=form.email.data).first()
        if user:
            token = user.generate_reset_token()
            send_email(user.email, 'Reset Your Password',
                       'auth/email/reset_password',
                       user=user, token=token,
                       next=request.args.get('next'))
        flash('An email with instructions to reset your password has been '
              'sent to you.')
        return redirect(url_for('auth.login'))
    return render_template('auth/reset_password.html', form=form)


@auth.route('/reset/<token>', methods=['GET', 'POST'])
def password_reset(token):
    if not current_user.is_anonymous:
        return redirect(url_for('main.index'))
    form = PasswordResetForm()
    if form.validate_on_submit():
        user = User.query.filter_by(email=form.email.data).first()
        if user is None:
            return redirect(url_for('main.index'))
        if user.reset_password(token, form.password.data):
            flash('Your password has been updated.')
            return redirect(url_for('auth.login'))
        else:
            return redirect(url_for('main.index'))
    return render_template('auth/reset_password.html', form=form)
```
### 用户添加重置密码功能
文件：**models.py**
```
def generate_reset_token(self, expiration=3600):
    s = Serializer(current_app.config['SECRET_KEY'], expiration)
    return s.dumps({'reset': self.id})

def reset_password(self, token, new_password):
    s = Serializer(current_app.config['SECRET_KEY'])
    try:
        data = s.loads(token)
    except:
        return False
    if data.get('reset') != self.id:
        return False
    self.password = new_password
    db.session.add(self)
    return True
```
### 邮件内容
文件：**app/templates/auth/email/reset_password.html**
```
<p>Dear {{ user.name }},</p>
<p>Welcome to <b>Harry's blog</b>!</p>
<p>To confirm your account please <a href="{{ url_for('auth.confirm', token=token, _external=True) }}">click here</a>.</p>
<p>Alternatively, you can paste the following link in your browser's address bar:</p>
<p>{{ url_for('auth.confirm', token=token, _external=True) }}</p>
<p>Sincerely,</p>
<p>Harry</p>
<p><small>Note: replies to this email address are not monitored.</small></p>
```
文件：**app/templates/auth/email/reset_password.html**
```
Dear {{ user.name }},

To reset your password click on the following link:

{{ url_for('auth.password_reset', token=token, _external=True) }}

If you have not requested a password reset simply ignore this message.

Sincerely,

Harry

Note: replies to this email address are not monitored.
```
## 修改邮件地址
修改邮件地址前，需要向新邮件地址发送确认邮件
### 添加表单
文件：**app/auth/forms.py**
```
class ChangeEmailForm(Form):
    email = StringField('New Email', validators=[Required(), Length(1, 64),
                                                 Email()])
    password = PasswordField('Password', validators=[Required()])
    submit = SubmitField('Update Email Address')

    def validate_email(self, field):
        if User.query.filter_by(email=field.data).first():
            raise ValidationError('Email already registered.')
```
### 用户模型添加更改邮件功能
文件：**models.py**
```
def generate_email_change_token(self, new_email, expiration=3600):
    s = Serializer(current_app.config['SECRET_KEY'], expiration)
    return s.dumps({'change_email': self.id, 'new_email': new_email})

def change_email(self, token):
    s = Serializer(current_app.config['SECRET_KEY'])
    try:
        data = s.loads(token)
    except:
        return False
    if data.get('change_email') != self.id:
        return False
    new_email = data.get('new_email')
    if new_email is None:
        return False
    if self.query.filter_by(email=new_email).first() is not None:
        return False
    self.email = new_email
    db.session.add(self)
    return True
```
### 编写页面
文件：**app/templates/auth/login.html**
```
{% extends "base.html" %}
{% import "bootstrap/wtf.html" as wtf %}

{% block title %}Change Email Address{% endblock %}

{% block page_content %}
<div class="page-header">
    <h1>Change Your Email Address</h1>
</div>
<div class="col-md-4">
    {{ wtf.quick_form(form) }}
</div>
{% endblock %}
```
### 添加改变邮件的标签
提供一个菜单，可以选择修改邮件地址
文件 **app/templates/base.html**
```
<li><a href="{{ url_for('auth.change_email_request') }}">Change Email</a></li>
```
### 编写视图函数
```
from .forms import ChangeEmailForm

@auth.route('/change-email', methods=['GET', 'POST'])
@login_required
def change_email_request():
    form = ChangeEmailForm()
    if form.validate_on_submit():
        if current_user.verify_password(form.password.data):
            new_email = form.email.data
            token = current_user.generate_email_change_token(new_email)
            send_email(new_email, 'Confirm your email address',
                       'auth/email/change_email',
                       user=current_user, token=token)
            flash('An email with instructions to confirm your new email '
                  'address has been sent to you.')
            return redirect(url_for('main.index'))
        else:
            flash('Invalid email or password.')
    return render_template("auth/change_email.html", form=form)


@auth.route('/change-email/<token>')
@login_required
def change_email(token):
    if current_user.change_email(token):
        flash('Your email address has been updated.')
    else:
        flash('Invalid request.')
    return redirect(url_for('main.index'))
```
### 编写邮件内容
文件：**app/templates/auth/email/change_email.html**
```
<p>Dear {{ user.name }},</p>
<p>To confirm your new email address <a href="{{ url_for('auth.change_email', token=token, _external=True) }}">click here</a>.</p>
<p>Alternatively, you can paste the following link in your browser's address bar:</p>
<p>{{ url_for('auth.change_email', token=token, _external=True) }}</p>
<p>Sincerely,</p>
<p>Harry</p>
<p><small>Note: replies to this email address are not monitored.</small></p>
```
文件：**app/templates/auth/email/change_email.txt**
```
Dear {{ user.name }},

To confirm your new email address click on the following link:

{{ url_for('auth.change_email', token=token, _external=True) }}

Sincerely,

Harry

Note: replies to this email address are not monitored.
```
