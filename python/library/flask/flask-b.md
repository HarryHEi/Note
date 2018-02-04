---
title: myblog-0b 关注
date: 2016-04-22 20:20:05
tags: [flask]
---

# 关注
## 多对多关系
一对多关系，一般是在‘多’的那一方添加一个外键连接
多对多关系需要添加第三张表，用来储存关系
文件：**app/models.py**
```
class Follow(db.Model):
    __tablename__ = 'follows'
    follower_id = db.Column(db.Integer, db.ForeignKey('users.id'),
                            primary_key=True)
    followed_id = db.Column(db.Integer, db.ForeignKey('users.id'),
                            primary_key=True)
    timestamp = db.Column(db.DateTime, default=datetime.utcnow)

class User(UserMixin, db.Model):

    #...
    followed = db.relationship('Follow',
                               foreign_keys=[Follow.follower_id],   #为了消除歧义指定外键
                               backref=db.backref('follower', lazy='joined'),
                               lazy='dynamic',
                               cascade='all, delete-orphan')
    followers = db.relationship('Follow',
                                foreign_keys=[Follow.followed_id],
                                backref=db.backref('followed', lazy='joined'),
                                lazy='dynamic',
                                cascade='all, delete-orphan')

    #...
    def follow(self, user):
        if not self.is_following(user):
            f = Follow(followed=user)
            self.followed.append(f)

    def unfollow(self, user):
        f = self.followed.filter_by(followed_id=user.id).first()
        if f:
            self.followed.remove(f)

    def is_following(self, user):
        return self.followed.filter_by(
            followed_id=user.id).first() is not None

    def is_followed_by(self, user):
        return self.followers.filter_by(
            follower_id=user.id).first() is not None
```
## 显示关注
在用户信息页面显示关注信息
文件：**app/templates/user.html**
```
<p>
    {% if current_user.can(Permission.FOLLOW) and user != current_user %}
        {% if not current_user.is_following(user) %}
        <a href="{{ url_for('.follow', name=user.name) }}" class="btn btn-primary">关注</a>
        {% else %}
        <a href="{{ url_for('.unfollow', name=user.name) }}" class="btn btn-default">取消关注</a>
        {% endif %}
    {% endif %}
    <a href="{{ url_for('.followers', name=user.name) }}">粉丝数：<span class="badge">{{ user.followers.count() }}</span></a>
    <a href="{{ url_for('.followed_by', name=user.name) }}">正在关注：<span class="badge">{{ user.followed.count() }}</span></a>
    {% if current_user.is_authenticated and user != current_user and user.is_following(current_user) %}
    | <span class="label label-default">关注了你</span>
    {% endif %}
</p>
```
### 关注的相应路由
文件：**app/main/views.py**
关注(取消关注)用户的路由：
```
@main.route('/follow/<name>')
@login_required
@permission_required(Permission.FOLLOW)
def follow(name):
    #确保用户存在
    user = User.query.filter_by(name=name).first()
    if user is None:
        flash('无效用户')
        return redirect(url_for('.index'))
    #确保用户没有关注该用户
    if current_user.is_following(user):
        flash('你已经关注了这个用户')
        return redirect(url_for('.user', name=name))
    current_user.follow(user)
    flash('成功关注 %s.' % name)
    return redirect(url_for('.user', name=name))

@main.route('/unfollow/<name>')
@login_required
@permission_required(Permission.FOLLOW)
def unfollow(name):
    user = User.query.filter_by(name=name).first()
    if user is None:
        flash('无效用户')
        return redirect(url_for('.index'))
    if not current_user.is_following(user):
        flash('你没有关注该用户')
        return redirect(url_for('.user', name=name))
    current_user.unfollow(user)
    flash('你已经取消关注 %s' % name)
    return redirect(url_for('.user', name=name))
```
### 显示关注者
关注数据从user.followd关系获取
FLASKY_FOLLOWERS_PER_PAGE在config.py默认配置为50
文件：**app/main/views.py**
```
@main.route('/followers/<name>')
def followers(name):
    user = User.query.filter_by(name=name).first()
    if user is None:
        flash('无效用户')
        return redirect(url_for('.index'))
    page = request.args.get('page', 1, type=int)
    pagination = user.followers.paginate(
        page, per_page=current_app.config['FLASKY_FOLLOWERS_PER_PAGE'],
        error_out=False)
    follows = [{'user': item.follower, 'timestamp': item.timestamp}
               for item in pagination.items]
    return render_template('followers.html', user=user, title="Followers of",
                           endpoint='.followers', pagination=pagination,
                           follows=follows)
```
## 关注者的博客文章
### 数据库查询
**Post.query.join(Follow, Follow.followed_id == Post.author_id)**联接follows和posts
(\*\*\*)**.filter(Follow.follower_id == self.id)**找到关注者为当前用户的值
类似的用mysql语句实现
```
mysql> select u.id,u.name,f.followed_id,p.body from users as u 
        ->  join follows as f on u.id=f.follower_id 
        ->  join posts as p on f.followed_id=p.author_id; 
```
在用户模型添加查询关注者文章功能
文件：**app/models.py**
```
class User(db.Model):
    # ...
    @property
    def followed_posts(self):
        return Post.query.join(Follow, Follow.followed_id == Post.author_id)\
            .filter(Follow.follower_id == self.id)
```
### 在首页显示所关注者文章
显示所有用户或者关注用户文章的flag保存在cookie(show_followed)中，使用两个路由设定值
Cookies 是设置在响应对象上的。由于通常视图函数只是返回字符串，之后 Flask 将字符串转换为响应对象。如果要显式地转换，你可以使用make_response() 函数然后再进行修改
文件：**app/main/views.py**
```
@main.route('/', methods=['GET', 'POST'])
def index():
    
    #...
    show_followed = False
    if current_user.is_authenticated:
        show_followed = bool(request.cookies.get('show_followed', ''))
    if show_followed:
        query = current_user.followed_posts
    else:
        query = Post.query
    pagination = query.order_by(Post.timestamp.desc()).paginate(
        page, per_page=current_app.config['FLASKY_POSTS_PER_PAGE'],
        error_out=False)
    posts = pagination.items
    return render_template('index.html', form=form, posts=posts,
                           show_followed=show_followed, pagination=pagination)

@main.route('/all')
@login_required
def show_all():
    resp = make_response(redirect(url_for('.index')))
    resp.set_cookie('show_followed', '', max_age=30*24*60*60)
    return resp


@main.route('/followed')
@login_required
def show_followed():
    resp = make_response(redirect(url_for('.index')))
    resp.set_cookie('show_followed', '1', max_age=30*24*60*60)
    return resp
```
在首页渲染两个标签对应两个路由
文件：**app/templates/index.html**
```
<div class="post-tabs">
    <ul class="nav nav-tabs">
        <li{% if not show_followed %} class="active"{% endif %}><a href="{{ url_for('.show_all') }}">All</a></li>
        {% if current_user.is_authenticated %}
        <li{% if show_followed %} class="active"{% endif %}><a href="{{ url_for('.show_followed') }}">Followers</a></li>
        {% endif %}
    </ul>
    {% include '_posts.html' %}
</div>
```
### 用户自己关注自己
添加一个函数，批量更新用户关注自己
文件：**app/models.py**
```
class User(UserMixin, db.Model):
    #...
    def __init__(self, **kwargs):
        #...
        self.followed.append(Follow(followed=self))

    @staticmethod
    def add_self_follows():
        for user in User.query.all():
            if not user.is_following(user):
                user.follow(user)
                db.session.add(user)
                db.session.commit()
```
在Shell界面调用
```
(venv)$ python manage.py shell
>>> User.add_self_follows()
```
静止用户取消关注自己
文件：app/main/views.py
```
@main.route('/unfollow/<name>')
@login_required
@permission_required(Permission.FOLLOW)
def unfollow(name):
    #...
    if user == current_user:
        flash('小伙子可以的，不过用户不能取消关注自己')
        return redirect(url_for('.index'))
```
