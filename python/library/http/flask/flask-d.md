---
title: myblog-0d API
date: 2016-04-23 17:00:37
tags: [flask]
---

# 应用编程接口
业务逻辑越来越多的移动到了客户端一侧，开创出一种称为RIA(富互联网应用 Rich Internet Application)的架构，服务器主要功能是提供数据存取服务。
在这种模式下，服务器变成了Web服务或者应用编程接口(API Application Programming Interface)。
## REST简介
REST(表现层状态转移, Representational State Transfer)架构特征

+ 客户端-服务器
客户端和服务器之间必须有明确的界线。
+ 无状态
客户端发出的请求中必须包含所有必要的信息。服务器不能在两次请求之间保存客户端的任何状态。
+ 缓存
服务器发出的响应可以标记为可缓存或不可缓存,这样出于优化目的,客户端(或客户端和服务器之间的中间服务)可以使用缓存。
+ 接口统一
客户端访问服务器资源时使用的协议必须一致,定义良好,且已经标准化。REST Web服务最常使用的统一接口是 HTTP 协议。
+ 系统分层
在客户端和服务器之间可以按需插入代理服务器、缓存或网关,以提高性能、稳定性和伸缩性。
+ 按需代码
客户端可以选择从服务器上下载代码,在客户端的环境中执行。
## 使用flask提供REST Web服务
### API蓝本结构
API包的名字中有一个版本号，如果需要向前兼容，需要添加一个版本号的不同的包
在API蓝本中，各资源在不同模块实现
```
|-myblog
    |-app/
        |-api_1_0
            |-__init__.py
            |-users.py
            |-posts.py
            |-comments.py
            |-authentication.py
            |-errors.py
            |-decorators.py
```
API蓝本构造文件
文件：**app/api_1_0/__init__.py**
```
from flask import Blueprint

api = Blueprint('api', __name__)

from . import authentication, posts, users, comments, errors
```
注册API蓝本
文件：**app/__init__.py**
```
def create_app(config_name):
    #...
    from .api_1_0 import api as api_1_0_blueprint
    app.register_blueprint(api_1_0_blueprint, url_prefix='/api/v1.0')
    #...
```
### 错误处理
在错误处理程序中根据客户端请求的格式改写响应，这种技术称为**内容协商**
它向Web服务端发送JSON格式响应
文件：**app/main/errors.py**
```
from flask import render_template, request, jsonify

from . import main

@main.app_errorhandler(404)
def page_not_found(e):
    if request.accept_mimetypes.accept_json and \
            not request.accept_mimetypes.accept_html:
        response = jsonify({'error': 'not found'})
        response.status_code = 404
        return response
    return render_template('404.html'), 404

@main.app_errorhandler(403)
def forbidden(e):
    if request.accept_mimetypes.accept_json and \
            not request.accept_mimetypes.accept_html:
        response = jsonify({'error': 'forbidden'})
        response.status_code = 403
        return response
    return render_template('403.html'), 403

@main.app_errorhandler(500)
def internal_server_error(e):
    if request.accept_mimetypes.accept_json and \
            not request.accept_mimetypes.accept_html:
        response = jsonify({'error': 'internal server error'})
        response.status_code = 500
        return response
    return render_template('500.html'), 500
```
API蓝本中错误处理程序，Web服务的视图函数可以调用这些辅助函数生成错误响应
文件：**app/api_1_0/errors.py**
```
from flask import jsonify

def bad_request(message):
    response = jsonify({'error': 'bad request', 'message': message})
    response.status_code = 400
    return response

def unauthorized(message):
    response = jsonify({'error': 'unauthorized', 'message': message})
    response.status_code = 401
    return response

def forbidden(message):
    response = jsonify({'error': 'forbidden', 'message': message})
    response.status_code = 403
    return response
```
### 使用 Flask-HTTPAuth 认证用户
Web浏览器外的客户端很难提供对cookie的支持
因为REST架构基于HTTP协议，所以发送密令的最佳方式是使用HTTP认证
在HTTP认证中，用户密令包含在请求的Authorization首部中
Flask-HTTPAuth扩展提供了一个便利的包装，可以把协议的细节隐藏在修饰器之中
类似于Flask-Login提供的login_required修饰器
因为这种用户认证方法只在API蓝本中使用，所以Flask-HTTPAuth扩展只在蓝本包中初始化
验证回调函数把通过认证的用户保存在Flask的全局对象g中
文件：**app/api_1_0/authentication.py**
```
from flask.ext.httpauth import HTTPBasicAuth
from flask import g, jsonify

from ..models import User, AnonymousUser
from . import api

auth = HTTPBasicAuth()

@auth.verify_password
def verify_password(email, password):
    if email == '': #匿名登录
        g.current_user = AnonymousUser()
        return True
    user = User.query.filter_by(email=email_or_token).first()
    if not user:
        return False
    g.current_user = user
    return user.verify_password(password)
```
如果认证密令不正确，服务器向客户端返回401错误，默认情况下，Flask-HTTPAuth自动生成这个状态码
为了和API返回的其他错误保持一致，可以自定义这个错误响应
由于这个蓝本的所有路由都要使用相同方式进行保护，可以在before_request处理程序中使用一次login_required修饰器，应用到整个蓝本
文件：**app/api_1_0/authentication.py**
```
from .errors import unauthorized, forbidden

@auth.error_handler
def auth_error():
    return unauthorized('Invalid credentials')

@api.before_request
@auth.login_required
def before_request():
    if not g.current_user.is_anonymous and \
            not g.current_user.confirmed:
        return forbidden('Unconfirmed account')
```
### 基于令牌的认证
客户端首先把登录密令发送给一个特殊的URL，从而生成认证令牌
一旦用户获得令牌，就可用令牌代替登录密令的认证请求，处于安全考虑，令牌有过期时间
User模型添加新方法，generate_auth_token()使用编码后的用户id字段值生成一个签名令牌，还指定了以秒为单位的过期时间
verify_auth_token()接收参数是一个令牌，如果令牌可用就返回对应的用户
文件：**app/models.py**
```
class User(UserMixin, db.Model):
    #....
    def generate_auth_token(self, expiration):
        s = Serializer(current_app.config['SECRET_KEY'],
                       expires_in=expiration)
        return s.dumps({'id': self.id}).decode('ascii')

    @staticmethod
    def verify_auth_token(token):
        s = Serializer(current_app.config['SECRET_KEY'])
        try:
            data = s.loads(token)
        except:
            return None
        return User.query.get(data['id'])
```
更改回调函数，支持令牌
文件：**app/api_1_0/authentication.py**
```
@auth.verify_password
def verify_password(email_or_token, password):
    if email_or_token == '':
        g.current_user = AnonymousUser()
        return True
    if password == '':
        g.current_user = User.verify_auth_token(email_or_token)
        g.token_used = True
        return g.current_user is not None
    user = User.query.filter_by(email=email_or_token).first()
    if not user:
        return False
    g.current_user = user
    g.token_used = False
    return user.verify_password(password)
```
生成认证令牌，为了避免客户端使用旧令牌申请新令牌,要在视图函数中检查 g.token_used 变量的值,如果使用令牌进行认证就拒绝请求。
这个视图函数返回 JSON 格式的响应,其中包含了过期时间为 1 小时的令牌。JSON 格式的响应也包含过期时间。
文件：**app/api_1_0/authentication.py**
```
@api.route('/token')
def get_token():
    if g.current_user.is_anonymous or g.token_used:
        return unauthorized('Invalid credentials')
    return jsonify({'token': g.current_user.generate_auth_token(
        expiration=3600), 'expiration': 3600})
```
### 资源和JSON的序列化转换
文件：**app/models.py**
```
class User(UserMixin, db.Model):
    #...
    def to_json(self):
        json_user = {
            'url': url_for('api.get_post', id=self.id, _external=True),
            'name': self.name,
            'member_since': self.member_since,
            'last_seen': self.last_seen,
            'posts': url_for('api.get_user_posts', id=self.id, _external=True),
            'followed_posts': url_for('api.get_user_followed_posts',
                                      id=self.id, _external=True),
            'post_count': self.posts.count()
        }
        return json_user

class Post(db.Model):
    #...
    def to_json(self):
        json_post = {
            'url': url_for('api.get_post', id=self.id, _external=True),
            'body': self.body,
            'body_html': self.body_html,
            'timestamp': self.timestamp,
            'author': url_for('api.get_user', id=self.author_id,
                              _external=True),
            'comments': url_for('api.get_post_comments', id=self.id,
                                _external=True),
            'comment_count': self.comments.count()
        }
        return json_post
```
从JSON格式数据创建一篇博客文章
文件：**app/models.py**
```
class Post(db.Model):
    #...
    @staticmethod
    def from_json(json_post):
        body = json_post.get('body')
        if body is None or body == '':
            raise ValidationError('post does not have a body')
        return Post(body=body)
```
文件：**app/exceptions.py**
```
class ValidationError(ValueError):
    pass
```
创建一个全局异常处理程序
文件：**app/api_1_0/errors.py**
```
@api.errorhandler(ValidationError)
def validation_error(e):
    return bad_request(e.args[0])
```
第一个路由处理获取文章集合的请求
第二个路由返回单个博客文章
文件：**app/api_1_0/posts.py**
```
@api.route('/posts/')
@auth.login_required
def get_posts():
    posts = Post.query.all()
    return jsonify({ 'posts': [post.to_json() for post in posts] })

@api.route('/posts/<int:id>')
def get_post(id):
    post = Post.query.get_or_404(id)
    return jsonify(post.to_json())
```
POST请求处理程序把一篇新博客文章插入数据库
文件：**app/api_1_0/posts.py**
```
@api.route('/posts/', methods=['POST'])
@permission_required(Permission.WRITE_ARTICLES)
def new_post():
    post = Post.from_json(request.json)
    post.author = g.current_user
    db.session.add(post)
    db.session.commit()
    return jsonify(post.to_json()), 201, \
        {'Location': url_for('api.get_post', id=post.id, _external=True)}
```
permission_required修饰器和程序中使用的类似，但会针对API蓝本进行自定义
文件：**app/api_1_0/decorators.oy**
```
from functools import wraps
from flask import g

from .errors import forbidden

def permission_required(permission):
    def decorator(f):
        @wraps(f)
        def decorated_function(*args, **kwargs):
            if not g.current_user.can(permission):
                return forbidden('Insufficient permissions')
            return f(*args, **kwargs)
        return decorated_function
    return decorator
```
PUT请求处理程序，修饰器用来检查用户是否有写博客文章的权限
文件：**app/api_1_0/posts.py**
```
@api.route('/posts/<int:id>', methods=['PUT'])
@permission_required(Permission.WRITE_ARTICLES)
def edit_post(id):
    post = Post.query.get_or_404(id)
    if g.current_user != post.author and \
            not g.current_user.can(Permission.ADMINISTER):
        return forbidden('Insufficient permissions')
    post.body = request.json.get('body', post.body)
    db.session.add(post)
    return jsonify(post.to_json())
```
API资源

/users/<int:id> | GET | 一个用户
---------------------|-------|------------
/users/<int:id>/posts/ | GET | 一个用户发布的博客文章
/users/<int:id>/timeline/ | GET | 一个用户所关注用户发布的文章
/posts/ | GET 、POST | 所有博客文章
/posts/<int:id> | GET 、PUT | 一篇博客文章
/posts/<int:id/>comments/ | GET 、POST | 一篇博客文章中的评论
/comments/ | GET | 所有评论
/comments/<int:id> | GET | 一篇评论

### 分页大型资源集合
分页文章资源，JSON格式响应中的posts字段依旧包含各篇文章，但这只是完整集合的一部分
文件：**app/api_1_0/posts.py**
```
@api.route('/posts/')
def get_posts():
    page = request.args.get('page', 1, type=int)
    pagination = Post.query.paginate(
        page, per_page=current_app.config['FLASKY_POSTS_PER_PAGE'],
        error_out=False)
    posts = pagination.items
    prev = None
    if pagination.has_prev:
        prev = url_for('api.get_posts', page=page-1, _external=True)
    next = None
    if pagination.has_next:
        next = url_for('api.get_posts', page=page+1, _external=True)
    return jsonify({
        'posts': [post.to_json() for post in posts],
        'prev': prev,
        'next': next,
        'count': pagination.total
    })
```
### 使用HTTPie测试Web服务
用户登录发送GET请求
```
(venv)$ http --json --auth <email>:<password> GET http://localhost:5000/api/v1.0/posts/
```
匿名用户发送GET请求
```
(venv)$ http --json --auth  :  GET http://localhost:5000/api/v1.0/posts/
```
使用POST请求添加一篇新文章
```
(venv)$ http --json --auth harryx520@qq.com:admin POST http://localhost:5000/api/v1.0/posts/ "body=lalala,post from json"
```
如果要使用认证令牌，可向/api/v1.0/token发送请求
```
(venv)$ http --json --auth <email>:<password> http://localhost:5000/api/v1.0/token
```
在有效时间内，这里是一小时，这个令牌可用于访问API
```
(venv)$ http --json --auth eyJhbG...:  GET http://localhost:5000/api/v1.0/posts/
```
