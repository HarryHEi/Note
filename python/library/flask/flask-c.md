---
title: myblog-0c 评论
date: 2016-04-23 15:04:00
tags: [flask]
---

# 评论
## 数据库
### 数据库关系
文章posts到评论comments是一对多的关系，一个文章有多个评论
用户users到评论comments是一对多关系，一个用户有多个评论

### 模型
添加一个评论模型，和文章模型相比多了一个disabled字段，用于管理员查禁不当言论
对Markdown的审核更加严格，删除与段落相关的标签
文件： `app/models.py`

```
class Comment(db.Model):
    __tablename__ = 'comments'
    id = db.Column(db.Integer, primary_key=True)
    body = db.Column(db.Text)
    body_html = db.Column(db.Text)
    timestamp = db.Column(db.DateTime, index=True, default=datetime.utcnow)
    disabled = db.Column(db.Boolean)
    author_id = db.Column(db.Integer, db.ForeignKey('users.id'))
    post_id = db.Column(db.Integer, db.ForeignKey('posts.id'))

    @staticmethod
    def on_changed_body(target, value, oldvalue, initiator):
        allowed_tags = ['a', 'abbr', 'acronym', 'b', 'code', 'em', 'i',
                        'strong']
        target.body_html = bleach.linkify(bleach.clean(
            markdown(value, output_format='html'),
            tags=allowed_tags, strip=True))

db.event.listen(Comment.body, 'set', Comment.on_changed_body)

class User(db.Model):
    # ...
    comments = db.relationship('Comment', backref='author', lazy='dynamic')

class Post(db.Model):
    # ...
    comments = db.relationship('Comment', backref='post', lazy='dynamic')
```

## 提交和显示评论
### 提交评论
创建一个表单
文件： `app/main/forms.py`

```
class CommentForm(Form):
    body = StringField('评论', validators=[Required()])
    submit = SubmitField('提交')
```
视图函数支持评论，和post视图函数相同，要使用`current_user._get_current_object()`获取用户对象
按照时间戳排序，新评论显示在底部

文件： `app/main/views.py`

```
@main.route('/post/<int:id>', methods=['GET', 'POST'])
def post(id):
    post = Post.query.get_or_404(id)
    form = CommentForm()
    if form.validate_on_submit():
        comment = Comment(body=form.body.data,
                          post=post,
                          author=current_user._get_current_object())
        db.session.add(comment)
        flash('评论成功')
        return redirect(url_for('.post', id=post.id, page=-1))
    page = request.args.get('page', 1, type=int)
    if page == -1:
        page = (post.comments.count() - 1) // \
            current_app.config['FLASKY_COMMENTS_PER_PAGE'] + 1
    pagination = post.comments.order_by(Comment.timestamp.asc()).paginate(
        page, per_page=current_app.config['FLASKY_COMMENTS_PER_PAGE'],
        error_out=False)
    comments = pagination.items
    return render_template('post.html', posts=[post], form=form,
                           comments=comments, pagination=pagination)
```

### 显示评论
渲染过程和_posts.html类似
文件： `app/templates/_comments.html`

```
<ul class="comments">
    {% for comment in comments %}
    <li class="comment">
        <div class="comment-thumbnail">
            <a href="{{ url_for('.user', name=comment.author.name) }}">
                <img class="img-rounded profile-thumbnail" src="{{ comment.author.avatar}}" width="40px" height="40px">
            </a>
        </div>
        <div class="comment-content">
            <div class="comment-date">{{ moment(comment.timestamp).fromNow() }}</div>
            <div class="comment-author"><a href="{{ url_for('.user', name=comment.author.name) }}">{{ comment.author.name }}</a></div>
            <div class="comment-body">
                {% if comment.body_html %}
                    {{ comment.body_html | safe }}
                {% else %}
                    {{ comment.body }}
                {% endif %}
            </div>
        </div>
    </li>
    {% endfor %}
</ul>
```

### 在博客文章添加评论链接
`#comments`链接到评论顶部
文件： `app/templates/_posts.html`

```
<a href="{{ url_for('.post', id=post.id) }}#comments">
    <span class="label label-primary">{{ post.comments.count() }} 评论</span>
</a>
```

文件： `app/templates/post.html`

```
{% extends "base.html" %}
{% import "bootstrap/wtf.html" as wtf %}
{% import "_macros.html" as macros %}

{% block title %}Post{% endblock %}

{% block page_content %}
{% include '_posts.html' %}
<h4 id="comments">评论</h4>
{% if current_user.can(Permission.COMMENT) %}
<div class="comment-form">
    {{ wtf.quick_form(form) }}
</div>
{% endif %}
{% include '_comments.html' %}
{% if pagination %}
<div class="pagination">
    {{ macros.pagination_widget(pagination, '.post', fragment='#comments', id=posts[0].id) }}
</div>
{% endif %}
{% endblock %}
```

## 管理评论
### 管理评论的链接
在导航条添加管理评论的链接
文件： `app/templates/base.html`

```
{% if current_user.can(Permission.MODERATE_COMMENTS) %}
<li><a href="{{ url_for('main.moderate') }}">管理评论</a></li>
{% endif %}
```

### 管理评论的路由
最近发布的评论会显示在最前面，每篇评论下都会有一个按钮，用来切换disabled的值
文件： `app/main/views.py`

```
@main.route('/moderate')
@login_required
@permission_required(Permission.MODERATE_COMMENTS)
def moderate():
    page = request.args.get('page', 1, type=int)
    pagination = Comment.query.order_by(Comment.timestamp.desc()).paginate(
        page, per_page=current_app.config['FLASKY_COMMENTS_PER_PAGE'],
        error_out=False)
    comments = pagination.items
    return render_template('moderate.html', comments=comments,
                           pagination=pagination, page=page)
```

### 渲染页面
将渲染工作交给_comments.html之前，设置moderate的值
文件： `app/templates/moderate.html`

```
{% extends "base.html" %}
{% import "_macros.html" as macros %}

{% block title %}管理评论{% endblock %}

{% block page_content %}
<div class="page-header">
    <h1>Comment Moderation</h1>
</div>
{% set moderate = True %}
{% include '_comments.html' %}
{% if pagination %}
<div class="pagination">
    {{ macros.pagination_widget(pagination, '.moderate') }}
</div>
{% endif %}
{% endblock %}
```
如果评论被设置为disable，将不能显示
文件： `app/templates/_comments.html`

```
<div class="comment-body">
    {% if comment.disabled %}
    <p><i>该评论无法显示</i></p>
    {% endif %}
    {% if comment.body_html %}
        {{ comment.body_html | safe }}
    {% else %}
        {{ comment.body }}
    {% endif %}
</div>
{% if moderate %}
    <br>
    {% if comment.disabled %}
    <a class="btn btn-default btn-xs" href="{{ url_for('.moderate_enable', id=comment.id, page=page) }}">Enable</a>
    {% else %}
    <a class="btn btn-danger btn-xs" href="{{ url_for('.moderate_disable', id=comment.id, page=page) }}">Disable</a>
    {% endif %}
{% endif %}
```
添加enable和disable评论的路由
文件： `app/main/views.py`

```
@main.route('/moderate/enable/<int:id>')
@login_required
@permission_required(Permission.MODERATE_COMMENTS)
def moderate_enable(id):
    comment = Comment.query.get_or_404(id)
    comment.disabled = False
    db.session.add(comment)
    return redirect(url_for('.moderate',
                            page=request.args.get('page', 1, type=int)))

@main.route('/moderate/disable/<int:id>')
@login_required
@permission_required(Permission.MODERATE_COMMENTS)
def moderate_disable(id):
    comment = Comment.query.get_or_404(id)
    comment.disabled = True
    db.session.add(comment)
    return redirect(url_for('.moderate',
                            page=request.args.get('page', 1, type=int)))
```
