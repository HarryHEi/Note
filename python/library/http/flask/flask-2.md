---
title: myblog-02 表单
date: 2016-04-12 20:41:49
tags: [flask]
---

# 表单
## 跨站请求伪造保护
使用**Flask-WTF**扩展可以把处理表单
默认情况下，Flask-WTF可以保护所有表单免收**CSRF**(Cross-Site Request Forgery)攻击
为了实现CSRF保护，Flask-WTF需要程序设置一个密钥
```
app.config['SECRET_KEY'] = 'hard to guess string'
```
## 表单类
定义一个表单类
```
from flask.ext.wtf import Form
from wtforms import StringField, SubmitField
from wtforms.validators import Required

class NameForm(Form):
    #StringField类表示属性type='text'的<input>元素
    #验证函数Required()确保提交的字段不为空
    name = StringField('name: ', validators=[Required()])
    #SubmitField类表示type='submit'的<input>元素
    submit = SubmitField('Submit')
```

## WTForms支持的HTML标准字段

字段类型 | 说  明
-------------|-----------
StringField | 文本字段
TextAreaField | 多行文本字段
PasswordField | 密码文本字段
HiddenField | 隐藏文本字段
DateField | 文本字段,值为 datetime.date 格式
DateTimeField | 文本字段,值为 datetime.datetime 格式
IntegerField | 文本字段,值为整数
DecimalField | 文本字段,值为 decimal.Decimal
FloatField | 文本字段,值为浮点数
BooleanField | 复选框,值为 True 和 False
RadioField | 一组单选框
SelectField | 下拉列表
SelectMultipleField | 下拉列表,可选择多个值
FileField | 文件上传字段
SubmitField | 表单提交按钮
FormField | 把表单作为字段嵌入另一个表单
FieldList | 一组指定类型的字段

## 表单渲染成HTML
使用Bootstrap中的表单样式
```
{% import "bootstrap/wtf.html" as wtf %}

{{ wtf.quick_form(form) }}
```
## 在视图函数中处理表单
视图函数需要渲染表单，并且要接收表单数据
```
#修饰器中添加Methods参数，视图函数注册为GET和POST请求的处理程序
@app.route('/', methods=['GET', 'POST'])
def index():
    #存放表单读取的名字
    name = None
    #NameForm实例来表示表单
    form = NameForm()
    #提交表单后如果能被验证函数接受，form.validate_on_submit()为True
    if form.validate_on_submit():
        name = form.name.data
        form.name.data = ''
    return render_template('index.html', form=form, name=name)
```
## "session会话"
可以把数据储存在**用户会话**(session)中
默认情况，用户会话保存在cookie中，使用SECRET_KEY进行加密签名
```
from flask import session,redirect,url_for

@app.route('/', methods=['GET', 'POST'])
def index():
    form = NameForm()
    if form.validate_on_submit():
        #变量值保存在会话
        session['name'] = form.name.data
        #redirect()函数的参数是重定向的URL
        #推荐使用url_for()
        return redirect(url_for('index'))
    return render_template('index.html', form=form, name=session.get('name'))
```
## redirect()重定向实现一个简单的登陆界面
```
@app.route('/', methods=['GET', 'POST'])
def index():
    form = NameForm()
    if form.validate_on_submit():
        session['name'] = form.name.data
        return redirect(url_for('login'))
    return render_template('index.html', form=form, name=session.get('name'))

@app.route('/nologin')
def nologin():
    abort(404)

@app.route('/login')
def login():
    if not session.get('name'):
        return redirect(url_for('nologin'))
    return render_template('user.html', name=session.get('name'))
```
## flash消息
调用**flash()**
```
from flask import url_for,flash

flash("I think you are new here~")
```
在模板中渲染flash消息
```
{% for message in get_flashed_messages() %}

  <div class="alert alert-warning">
    <button type="button" class="close" data-dismiss="alert">&times;</button>
    {{ message }}
  </div>

{% endfor %}
```
