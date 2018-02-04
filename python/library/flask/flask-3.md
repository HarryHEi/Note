---
title: myblog-03 数据库和用户认证
date: 2016-04-13 14:07:25
tags: [flask,mysql]
---

# Flask-SQLAlchemy管理数据库
## 初始化一个数据库
```
from flask.ext.sqlalchemy import SQLAlchemy

app.config["SQLALCHEMY_TRACK_MODIFICATIONS"] = True
app.config['SQLALCHEMY_DATABASE_URI'] ='mysql://username:password@hostname/database'
app.config['SQLALCHEMY_COMMIT_ON_TEARDOWN'] = True

db = SQLAlchemy(app)
```
## 定义模型
模型表示程序使用的持久化实体，在**ORM**（对象关系映射：Object Relation Mapping）中，模型一般是一个python类
```
class User(db.Model):
    __tablename__ = 'users'
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(20), unique=True)

    def __repr__(self):
        return '<User %r>' % self.name
```
[详细指令](http://harryhei.github.io/2016/03/12/flask-mysql/)
## 数据库创建一个表
```
mysql> create table users(
    -> id int(32) primary key auto_increment,
    -> name char(20) unique);
```
## 插入一个数据
```
mysql> insert users
    -> (id,name)
    -> values (1,harry);
```
## 集成Python Shell
注册一个**make_shell_context()**回调函数
不用每次都从Shell导入模型和实例
```
from flask.ext.script import Shell

def make_shell_context():
    return dict(app=app, db=db, User=User)
manager.add_command("shell", Shell(make_context=make_shell_context))
```
## 视图函数操作
```
@app.route('/', methods=['GET', 'POST'])
def index():
    form = NameForm()
    if form.validate_on_submit():
        user = User.query.filter_by(name=form.name.data).first()
        if user is None:
            flash("I think you are new here~")
            user = User(name = form.name.data)
            db.session.add(user)
        session['name'] = form.name.data
        form.name.data = ''
        return redirect(url_for('index'))
    return render_template('index.html', form=form, name=session.get('name'))
```
## 创建迁移仓库
### 配置Flask-Migrate
Flask-Migrate提供了一个MigrateCommand类，可以附加到Flask-Script的manager对象上
```
from flask.ext.migrate import Migrate, MigrateCommand

# ...

migrate = Migrate(app, db)
manager.add_command('db', MigrateCommand)
```
### 初始化
如果已经初始化过，需要删除之前的版本，并且要删除之前创建migrations文件
```
mysql > DROP TABLE alembic_version;
```
初始化会创建migrations文件，所有迁移脚本都存放其中
```
(venv)$ python blog.py db init
  Creating directory /home/herui/myblog/migrations ... done
  Creating directory /home/herui/myblog/migrations/versions ... done
  Generating /home/herui/myblog/migrations/alembic.ini ... done
  Generating /home/herui/myblog/migrations/env.pyc ... done
  Generating /home/herui/myblog/migrations/script.py.mako ... done
  Generating /home/herui/myblog/migrations/env.py ... done
  Generating /home/herui/myblog/migrations/README ... done
  Please edit configuration/connection/logging settings in
  '/home/herui/myblog/migrations/alembic.ini' before proceeding.
```
### 创建迁移脚本
```
(venv)$ python blog.py db migrate -m "initial migration"
INFO  [alembic.runtime.migration] Context impl MySQLImpl.
INFO  [alembic.runtime.migration] Will assume non-transactional DDL.
INFO  [alembic.env] No changes in schema detected.
```
### 更新数据库
```
(venv)$ python blog.py db upgrade
INFO  [alembic.runtime.migration] Context impl MySQLImpl.
INFO  [alembic.runtime.migration] Will assume non-transactional DDL.
```
# 用户认证
## 密码散列值
使用Werkzeug实现密码散列

+ **generate_password_hash(password, method= pbkdf2:sha1 , salt_length=8)**：
这个函数将原始密码作为输入，以字符串形式输出密码散列值，method，salt_length默认值能满足大部分需求
+ **check_password_hash(hash, password)**：
这个函数参数是从数据库读回的密码散列值和用户输入的密码，返回值为True表示密码正确

生成散列值后不能还原成原来的密码
在User模型中加入密码散列
```
from werkzeug.security import generate_password_hash, check_password_hash

@property
def password(self):
    raise AttributeError('password is not a readable attribute')

@password.setter
def password(self, password):
    self.password_hash = generate_password_hash(password)

def verify_password(self, password):
    return check_password_hash(self.password_hash, password)
```
## 创建认证蓝本
对于不同的程序功能使用不同的蓝本
文件：**app/auth/__init__.py**
```
from flask import Blueprint

auth = Blueprint('auth', __name__)

from . import views
```
添加路由和视图函数
文件：**app/auth/views.py**
```
from flask import render_template

from . import auth

@auth.route('/login')
def login():
    return render_template('auth/login.html')
```
文件：**app/auth/login.html**
```
{% extends "base.html" %}

{% block title %}login{% endblock %}

{% block page_content %}
<div class="page-header">
    <h1>Login</h1>
</div>
{% endblock %}
```
在**create_app()**工厂函数中附加auth蓝本
url_prefix是可选参数，如果使用了这个参数，注册后蓝本中定义的所有路由都会加上前缀，这个例子为即/auth
文件：**app/__init__.py**
```
def create_app(config_name):

    #...

    from .auth import auth as auth_blueprint
    app.register_blueprint(auth_blueprint, url_prefix='/auth')

    return app
```
## 使用Flask-Login认证用户
### 准备用于登录的用户模型
Flask-Login提供了一个UserMixin类，包含一些方法的默认实现
文件：app/models.py
```
from flask.ext.login import UserMixin

class User(UserMixin, db.Model):
    __tablename__ = 'users'
    id = db.Column(db.Integer, primary_key = True)
    email = db.Column(db.String(64), unique=True, index=True)
    name = db.Column(db.String(64), unique=True, index=True)
    password_hash = db.Column(db.String(128))
```
### Flask-Login在工厂函数中初始化
login_manager对象的session_protection可以设置为None、'basic'、'strong'，以提供不同的安全等级防止用户会话遭篡改
Flask-Login会记录客户端IP地址和浏览器的用户代理信息，如果发现异动就登出用户
login_view属性设置登录页面的端点，登录路由在蓝本中定义，要在前面加上蓝本名字(auth)
文件：**app/__init__.py**
```
from flask.ext.login import LoginManager

login_manager = LoginManager()
login_manager.session_protection = 'strong'
login_manager.login_view = 'auth.login'

def create_app(config_name):
    # ...
    login_manager.init_app(app)
    # ...
```
回调函数，使用指定标识符加载用户，返回用户对象或者None
文件：**app/models.py**
```
from . import login_manager

@login_manager.user_loader
def load_user(user_id):
    return User.query.get(int(user_id))
```
### 保护路由
为了保护路由只让认证用户访问，Flask-Login提供了一个login_require修饰器
如果没有认证的用户访问这个路由，会拦截请求，把用户发往登录页面
用法如下
```
from flask.ext.login import login_required

@app.route('/secret')
@login_required
def secret():
    return 'Only authenticated users are allowed!'
```
### 添加登录表单
包含输入邮件、密码、记住无复选框和提交按钮
文件：**app/auth/forms.py**
```
from flask.ext.wtf import Form
from wtforms import StringField, PasswordField, BooleanField, SubmitField
from wtforms.validators import Required, Length, Email

class LoginForm(Form):
    email = StringField('Email', validators=[Required(), Length(1, 64),Email()])
    password = PasswordField('Password', validators=[Required()])
    remember_me = BooleanField('Keep me logged in')
    submit = SubmitField('Log In')
```
### 登录页面：auth/login.html
```
{% extends "base.html" %}
{% import "bootstrap/wtf.html" as wtf %}

{% block title %}Login{% endblock %}

{% block page_content %}
<div class="page-header">
    <h1>Login</h1>
</div>
<div class="col-md-4">
    {{ wtf.quick_form(form) }}
</div>
{% endblock %}
```
### 在导航条中添加Sign In和Sign Out链接
```
<ul class="nav navbar-nav navbar-right">
{ % if current_user.is_authenticated() % }
<li><a href="{{ url_for('auth.logout') }}">Sign Out</a></li>
{ % else % }
<li><a href="{{ url_for('auth.login') }}">Sign In</a></li>
{ % endif % }
</ul>
```
### 登入用户
文件：**app/auth/views.py**
```
from flask import render_template, redirect, request, url_for, flash
from flask.ext.login import login_user, logout_user

from . import auth
from ..models import User
from .forms import LoginForm

@auth.route('/login', methods=['GET', 'POST'])
def login():
    #创建一个**LoginForm()**对象
    form = LoginForm()
    if form.validate_on_submit():
        #如果电子邮件对应的用户存在
        user = User.query.filter_by(email=form.email.data).first()

        调用用户对象的verify_password()方法，检测输入的密码
        if user is not None and user.verify_password(form.password.data):
            #调用Flask-Login的login_user函数，把用户标记为已登录，第二个参数为“记住我”
            login_user(user, form.remember_me.data)
            return redirect(request.args.get('next') or url_for('main.index'))
        flash('Invalid username or password.')

    #请求类型为GET时直接显示表单
    return render_template('auth/login.html', form=form)
```
### 登出用户
文件：**app/auth/views.py**
```
from flask.ext.login import logout_user, login_required

@auth.route('/logout')
@login_required
def logout():
    #调用Flask-Login中的logout_user()函数
    logout_user()
    flash('You have been logged out.')
    return redirect(url_for('main.index'))
```
### 测试登录
在Shell界面注册用户
```
(venv)$ python manage.py shell
>>> u = User(id=1,name="harry",email="harryx520@qq.com",password="lalala")
>>> db.session.add(u)
>>> db.session.commit()
```
查看数据库
```
mysql> select * from users;

|  1 | harryx520@qq.com | harry | pbkdf2:sha1:1(...)ffba4781 |
```

# 数据库常用操作

## 创建表

创建普通有一个约束的表

`person_id`作为主键约束

```
mysql> create table person
    -> (person_id int(5) unsigned,
    -> name char(64),
    -> gender enum('M','F'),
    -> state char(64),
    -> constraint pk_person primary key (person_id))default charset=utf8;
```

创建具有外键约束的表

`person_id`和`food`同时作为主键约束，`person_id`作为外键约束

```
mysql> create table fav_food
    -> (person_id int(5) unsigned,
    -> food char(64),
    -> constraint pk_fav_food primary key (person_id, food),
    -> constraint fk_fav_food_person_id foreign key (person_id)
    ->      references person (person_id))default charset=utf8;
```

## 修改表

alter table用于修改已经存在的表定义

```
mysql> alter table person modify person_id int(5) unsigned auto_increment;
```

## 插入数据

insert语句由三个部分构成：表的名称、列的名称、值

```
mysql> insert into person
    -> (name, gender, state)
    -> values ("harryx", "M", "中国");
```

## 查询数据

可以指定列、指定顺序、设置条件

可以添加如下：

+ 字符，比如数字和字符串
+ 表达式
+ 调用内建函数
+ 自定义函数

```
mysql> select person_id,name,gender from person where state="中国";

mysql> select * from person order by name;

mysql> select name,"is",gender from person;

mysql> select version(),user(),database();
```

### 使用别名

```
mysql> select name as n,gender as g from person;
```

### 去除重复

注意，需要先排序

```
mysql> select distinct person_id from fav_food;
```

查询的表可以是临时表和虚拟表

在子表查询

```
mysql> select e.name,e.state from (select * from person) as e;
```

### 视图

试图并不存储数据，只提供查询

```
mysql> create view person_vw as
    -> select name from person;
```

### 表连接

内连接（不用inner关键字的话也是默认内连接）
不提供`on`关键字的话是笛卡尔积

```
mysql> select p.name,f.food
    -> from person as p inner join fav_food as f
    -> on p.person_id = f.person_id;
```

如果如果连接的表关键字是相同的可以使用`using`代替`on`

```
mysql> select p.name,p.state,f.food
    -> from person as p inner join fav_food as f
    -> using (person_id);
```

连接多个表

```
mysql> select p.name,f.food,a.about
    -> from person as p inner join fav_food as f
    -> using(person_id)
    -> inner join about_me as a
    -> using(person_id);
```

### 分组

group by 根据某一列的值分组

having 对数据进行过滤

```
mysql> select p.name,count(f.food) 
    -> from person as p inner join fav_food as f 
    -> on p.person_id = f.person_id 
    -> group by p.name;
    -> having p.name != "harry";
```

### 排序

order by 用于对结果的列数据或者根据表达式结果排序

```
//根据state排序
mysql> select * from person order by state;

//根据state 和 name 排序
mysql> select * from person order by state,name;
```

加上desc降序排序

```
//降序排序
mysql> select * from person order by person_id desc;
```

limit 限制显示行数

```
//person_id最大的显示前三行
mysql> select * from person order by person_id desc limit 3;
```

### 使用集合

union 连接数据，排除重复项
union all 连接数据，保留重复项

intersect 交集，MySQL不支持

except 差集，MySQL不支持

## 更新数据

更新语句可以更新一行或者匹配到的多行

```
mysql> update person
    -> set state="美国"
    -> where person_id=4;
```

级联更新

```
mysql> alter table fav_food
    -> add constraint fk_food_peron foreign key (person_id)
    -> references person(person_id)
    -> on update cascade;
```

## 删除数据

如果被删除的列设置了外键，需要在子表添加外键时申明级联删除
在申明级联删除前先删除外键

```
mysql> alter table fav_food 
    -> add constraint fk_person_id foreign key (person_id) 
    -> references person (person_id) on delete cascade;

mysql> delete from person where name="o";
```

## XML

可以获取XML格式的数据，需要制定xml格式进入

```
mysql -u flask -p --xml flaskdb
```

## 事务

关闭自动提交

```
mysql> set autocommit=0;
```
 
关闭自动提交后，所有的SQL命令都在一个事务内
显式使用`rollback`回滚数据
显式使用`commit`提交数据

```
mysql> delete from person where person_id=4;
mysql> rollback;
```

设置保存点

保存点拥有名字，可以拥有多个保存点

```
mysql> delete from fav_food where food="ice";
mysql> savepoint point_1;
mysql> delete from fav_food where food="apple";
mysql> savepoint point_2;
mysql> rollback to savepoint point_1;
```

## 索引

添加索引

```
mysql> alter table person
    -> add index ind_name (name);
```

或者

```
mysql> create index ind_name on person(name);
```

查询索引

```
mysql> show index from person \G
```

删除某个索引

```
mysql> alter table person
    -> drop index ind_name;
```

或者

```
mysql> drop index ind_name on person;
```
