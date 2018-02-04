---
title: myblog-0a 文章链接和编辑
date: 2016-04-21 11:20:06
tags: [flask]
---

# 添加博客文章的固定链接
## 文章路由
到文章页面**post.html**的路由
文件：**app/main/views.py**
```
@main.route('/post/<int:id>')
def post(id):
    post = Post.query.get_or_404(id)
    return render_template('post.html', posts=[post])
```
## 文章模板
每个文章后面都会添加一个单独的链接
文件：**app/templates/_posts.html**
```
<ul class="posts">
    {% for post in posts %}
    #...
            <div class="post-footer">
                <a href="{{ url_for('.post', id=post.id) }}">
                    <span class="label label-default">more</span>
                </a>
            </div>
        </div>
    </li>
    {% endfor %}
</ul>
```
## 单独博客文章页面
从视图函数获得posts参数
文件：**app/templates/post.html**
```
{% extends "base.html" %}
{% import "_macros.html" as macros %}

{% block title %}Post{% enm dblock %}

{% block page_content %}
{% include '_posts.html' %S}
{% endblock %}
```
# 文章编辑器
## 渲染模板
文件：**app/templates/edit_post.html**
```
{% extends "base.html" %}
{% import "bootstrap/wtf.html" as wtf %}

{% block title %}Edit Post{% endblock %}

{% block page_content %}
<div class="page-header">
    <h1>编辑文章</h1>
</div>
<div>
    {{ wtf.quick_form(form) }}
</div>
{% endblock %}

{% block scripts %}
{{ super() }}
{{ pagedown.include_pagedown() }}
{% endblock %}
```
## 视图函数
只允许作者本人或者管理员编辑文章，如果其他人视图编辑，返回403
文件：**app/main/views.py**
```
@main.route('/edit/<int:id>', methods=['GET', 'POST'])
@login_required
def edit(id):
    post = Post.query.get_or_404(id)
    if current_user != post.author and \
            not current_user.can(Permission.ADMINISTER):
        abort(403)
    form = PostForm()
    if form.validate_on_submit():
        post.body = form.body.data
        db.session.add(post)
        flash('The post has been updated.')
        return redirect(url_for('.post', id=post.id))
    form.body.data = post.body
    return render_template('edit_post.html', form=form)
```
## 添加链接
文件：**app/templates/_posts.html**
```
<div class="post-footer">
    {% if current_user == post.author %}
    <a href="{{ url_for('.edit', id=post.id) }}">
        <span class="label label-primary">编辑</span>
    </a>
    {% elif current_user.is_administrator() %}
    <a href="{{ url_for('.edit', id=post.id) }}">
        <span class="label label-danger">编辑</span>
    </a>
    {% endif %}
    <a href="{{ url_for('.post', id=post.id) }}">
        <span class="label label-default">--more--</span>
    </a>
</div>
```
