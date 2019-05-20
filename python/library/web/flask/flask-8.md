---
title: myblog-08 博客文章
date: 2016-04-18 09:45:56
tags: [flask]
---

# 博客文章
## 文章支持
### 更新数据库模型
这里需要新建一个数据库模型，用来储存博客文章
用户与博客文章是一对多的关系
文件：**app/models.py**
```
class Post(db.Model):
    __tablename__ = "posts"
    id = db.Column(db.Integer, primary_key=True)
    body = db.Column(db.Text())
    timestamp = db.Column(db.DateTime(), index=True, default=datetime.utcnow)
    author_id = db.Column(db.Integer, db.ForeignKey(users.id))

class User(UserMixin, db.Model):
    
    #...
    posts = db.relationship('Post', backref='author', lazy='dynamic')
```
### 简历表单
建立一个表单，提供给用户写博客文章
文件：**app/main/forms.py**
```
class PostForm(Form):
    name = StringField('write something here: ', validators=[Required()])
    submit = SubmitField('Submit')
```
首页路由提供用户写文章的表单，并把文章上传到数据库，前提是用户拥有写文章的权限
用户第一次GET请求会把所有文章按照时间检索出来
用户对象的_get_current_object()类，是真正的用户对象
文件：**app/main/views.py**
```
@main.route('/', methods=['GET', 'POST'])
def index():
    form =PostForm()
    if current_user.can(Permission.WRITE_ARTICLES) adn \
            form.validate_on_submit():
        post = Post(body=form.body.data,
                            author=current_user._get_current_object())
        db.session.add(post)
        return redirect(url_for('.index'))
    posts = Post.query.order_by(Post.timestamp.desc()).all()
    return render_template('index.html', form=form, posts=posts)
```
### 显示文章
在主页渲染
利用循环，将所有内容显示出来
文件：**app/templates/index.html**
```
{% extends "base.html" %}
{% import "bootstrap/wtf.html" as wtf %}

{% block title %}主站{% endblock %}

{% block page_content %}
<div class="page-header">
    <h1>Hello, {% if current_user.is_authenticated %}{{ current_user.name }}{% else %}Stranger{% endif %}!</h1>
</div>
<div>
    {% if current_user.can(Permission.WRITE_ARTICLES) %}
    {{ wtf.quick_form(form) }}
    {% endif %}
</div>
<ul class="posts">
    {% for post in posts %}
    <li class="post">
        <div class="post-thumbnail">
            <a href="{{ url_for('.user', name=post.author.name) }}">
                <img class="img-rounded profile-thumbnail" src="{{ post.author.avatar}}" width="40px" height="40px">
            </a>
        </div>
        <div class="post-content">
            <div class="post-date">{{ moment(post.timestamp).fromNow() }}</div>
            <div class="post-author"><a href="{{ url_for('.user', name=post.author.name) }}">{{ post.author.name }}</a></div>
            <div class="post-body">{{ post.body }}</div>
        </div>
    </li>
    {% endfor %}
</ul>
{% endblock %}
```
### 添加CSS样式
添加一个css文件，保存在static文件夹里面
文件：**app/static/styles.css**
```
.profile-thumbnail {
    position: absolute;
}
.profile-header {
    min-height: 260px;
    margin-left: 280px;
}
ul.posts {
    list-style-type: none;
    padding: 0px;
    margin: 16px 0px 0px 0px;
    border-top: 1px solid #e0e0e0;
}
ul.posts li.post {
    padding: 8px;
    border-bottom: 1px solid #e0e0e0;
}
ul.posts li.post:hover {
    background-color: #f0f0f0;
}
div.post-date {
    float: right;
}
div.post-author {
    font-weight: bold;
}
div.post-thumbnail {
    position: absolute;
}
div.post-content {
    margin-left: 48px;
    min-height: 48px;
}
```
在base.html中导入css文件
文件：**app/templates/base.html**
```
{% block head %}
{{ super() }}
<link rel="icon" href="{{ url_for('static', filename='shortcut.ico') }}" type="image/x-icon">
<link rel="stylesheet" type="text/css" href="{{ url_for('static', filename='styles.css') }}">
{% endblock %}
```
