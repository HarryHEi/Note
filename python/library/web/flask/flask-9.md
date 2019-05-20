---
title: myblog-09 分页显示和markdown支持
date: 2016-04-19 08:41:49
tags: [flask]
---

# 分页显示微博
## 创建虚拟博客文章记录
### 环境依赖
自动生成依赖文件
```
(venv)$ pip freeze > requirements/common.txt 
```
配置环境
```
(venv) $ pip install -r requirements.txt
```
针对开发环境
文件：**requirements/dev.txt**
```
-r common.txt
ForgeryPy==0.1
```
### 添加创建虚拟用户功能
随机生成用户和其他数据通过forgery_py随机信息生成器生成
因为是随机生成，所以有可能重复，当重复时，会报出IntegrityError异常
检测出异常需要回滚数据
随机生成文章时，通过随机定义一个不超过用户个数的偏移量，检索随机用户
文件：**app/models.py**
```
class User(UserMixin, db.Model):

    #...
    @staticmethod
    def generate_fake(count=100):
        from sqlalchemy.exc import IntegrityError
        from random import seed
        import forgery_py

        seed()
        for i in range(count):
            u = User(email=forgery_py.internet.email_address(),
                     name=forgery_py.internet.user_name(True),
                     password=forgery_py.lorem_ipsum.word(),
                     confirmed=True,
                     location=forgery_py.address.city(),
                     about_me=forgery_py.lorem_ipsum.sentence(),
                     member_since=forgery_py.date.date(True))
            db.session.add(u)
            try:
                db.session.commit()
            except IntegrityError:
                db.session.rollback()

class Post(db.Model):

    #...
    @staticmethod
    def generate_fake(count=100):
        from random import seed, randint
        import forgery_py

        seed()
        user_count = User.query.count()
        for i in range(count):
            u = User.query.offset(randint(0, user_count - 1)).first()
            p = Post(body=forgery_py.lorem_ipsum.sentences(randint(1, 5)),
                     timestamp=forgery_py.date.date(True),
                     author=u)
            db.session.add(p)
            db.session.commit()
```
在shell下添加虚拟用户和文章
```
(venv) $ python manage.py shell
>>> User.generate_fake(100)
>>> Post.generate_fake(100)
```

## 数据库分页读取
获取页面的页数从request.args.get()中获取，如果没有明确指定，page默认为1，即第一页
在URL最后加上**?page=2**，page值为2，即为第二页
在order_by检索完之后，在进行paginate检索
第一个参数page为必须参数
第二个参数per_page为每页个数，在config.py里面配置成默认10
第三个参数为True时，如果请求的页数超出了范围会返回404，False返回空列表
文件：**app/main/views.py**
```
@main.route('/', methods=['GET', 'POST'])
def index():

    #...
    page = request.args.get('page', 1, type=int)
    pagination = Post.query.order_by(Post.timestamp.desc()).paginate(
        page, per_page=current_app.config['FLASKY_POSTS_PER_PAGE'],
        error_out=False)
    posts = pagination.items
    return render_template('index.html', form=form, posts=posts,
                           pagination=pagination)
```

## 添加分页导航
### Pagination属性
**paginate()**方法返回的是一个**Pagination**类对象，具有以下属性

属  性 | 说  明
--------|--------
items | 当前页面中的记录
query | 分页的源查询
page | 当前页数
prev_num |上一页的页数
next_num | 下一页的页数
has_next | 如果有下一页,返回 True
has_prev | 如果有上一页,返回 True
pages | 查询得到的总页数
per_page | 每页显示的记录数量
total | 查询返回的记录总数

可以调用的方法

+ iter_pages(left_edge=2,left_current=2,right_current=5,right_edge=2)：
一个迭代器,返回一个在分页导航中显示的页数列表。这个列表的最左边显示 left_edge页,当前页的左边显示left_current 页,当前页的右边显示 right_current 页,最右边显示 right_edge 页。
例如,在一个 100 页的列表中,当前页为第50 页,使用默认配置,这个方法会返回以下页数:1、2、 None 、48、49、50、51、52、53、54、55、 None 、99、100。 None 表示页数之间的间隔
+ prev()：上一页的分页对象
+ next()：下一页的分页对象

### 分页模块宏
检测当前页数是不是第一页或者最后一页，在上一页和下一页添加disabled类
文件保存在_macros.html中，在index和user中导入
文件：**app/templates/_macros.html**
```
{% macro pagination_widget(pagination, endpoint) %}
<ul class="pagination">
    <li{% if not pagination.has_prev %} class="disabled"{% endif %}>
        <a href="{% if pagination.has_prev %}{{ url_for(endpoint, page=pagination.prev_num, **kwargs) }}{% else %}#{% endif %}">
            &laquo;
        </a>
    </li>
    {% for p in pagination.iter_pages() %}
        {% if p %}
            {% if p == pagination.page %}
            <li class="active">
                <a href="{{ url_for(endpoint, page = p, **kwargs) }}">{{ p }}</a>
            </li>
            {% else %}
            <li>
                <a href="{{ url_for(endpoint, page = p, **kwargs) }}">{{ p }}</a>
            </li>
            {% endif %}
        {% else %}
        <li class="disabled"><a href="#">&hellip;</a></li>
        {% endif %}
    {% endfor %}
    <li{% if not pagination.has_next %} class="disabled"{% endif %}>
        <a href="{% if pagination.has_next %}{{ url_for(endpoint, page=pagination.next_num, **kwargs) }}{% else %}#{% endif %}">
            &raquo;
        </a>
    </li>
</ul>
{% endmacro %}
```
文件：**app/templates/index.html**
```
{% extends "base.html" %}
{% import "bootstrap/wtf.html" as wtf %}
{% import "_macros.html" as macros %}

#...
{% if pagination %}
<div class="pagination">
    {{ macros.pagination_widget(pagination, '.index') }}
</div>
{% endif %}
{% endblock %}
```
文件：**app/templates/user.html**
```
{% extends "base.html" %}
{% import "_macros.html" as macros %}

#...
<h3>{{ user.name }}的文章</h3>
{% include '_posts.html' %}
{% if pagination %}
<div class="pagination">
    {{ macros.pagination_widget(pagination, '.user', username=user.username) }}
</div>
{% endif %}
{% endblock %}
```
# Markdown和Flask-PageDown支持
## 需要的包
+ PageDown：使用JavaScript实现的客户端Markdown到HTML转换程序
+ Flask-PageDown：为Flask包装的PageDown，把PageDown集成到Flask-WTF表单
+ Markdown：使用 Python 实现的服务器端 Markdown 到 HTML 的转换程序
+ Bleach：使用 Python 实现的 HTML 清理器。

### 使用Flask-PageDown扩展
初始化
文件：**app/__init__.py**
```
#!/usr/bin/env python
#-*-coding:utf-8-*-

from flask.ext.pagedown import PageDown
#...

pagedown = PageDown()
#...
def create_app(config_name):
    
    #...
    pagedown.init_app(app)
    #...
```
添加Markdown表单
文件：**app/main/forms.py**
```
from flask.ext.pagedown.fields import PageDownField 

class PostForm(Form):
    body = PageDownField('写点什么', validators=[Required()])
    submit = SubmitField('提交')
```
Markdown预览，在index.html文件导入这个功能
文件：**app/templates/index.html**
```
{% block scripts %}
{{ super() }}
{{ pagedown.include_pagedown() }}
{% endblock %}
```
在用户模型中添加**body_html**储存Markdown转化成的HTML
**markdown()**函数把Markdown转化成HTML，然后把结果传给**clean()**，删除不在白名单的标签
attributes参数能够保存标签的属性，不然无法插入图片
最后，**linkify()**函数把纯文本中的URL转成适当的<a\>链接
数据库监听body字段'set'事件，如果发生变化，重新进行转换
文件：**app/models.py**
```
from markdown import markdown
import bleach

class Post(db.Model):

    #...
    body_html = db.Column(db.Text)

    #...
    @staticmethod
    def on_changed_body(target, value, oldvalue, initiator):
        allowed_tags = ['a', 'abbr', 'acronym', 'b', 'blockquote', 'code',
                        'em', 'i', 'li', 'ol', 'pre', 'strong', 'ul',
                        'h1', 'h2', 'h3', 'p', 'img']
        attrs = {
            "img":["src"]
        }
        target.body_html = bleach.linkify(bleach.clean(
            markdown(value, output_format='html'),
            tags=allowed_tags, strip=True ,attributes=attrs))

db.event.listen(Post.body, 'set', Post.on_changed_body)
```
渲染HTML格式内容时使用 | safe后缀，告诉Jinja2不要转义HTML元素
文件：**app/templates/__posts.html**
```
<div class="post-body">
    {% if post.body_html %}
        {{ post.body_html | safe }}
    {% else %}
        {{ post.body }}
    {% endif %}
</div>
```
