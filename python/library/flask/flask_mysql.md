---
title: flask_mysql
date: 2016-03-12 10:52:24
tags: [flask,mysql]
---

# 配置
```
app.config["SQLALCHEMY_TRACK_MODIFICATIONS"] = True
app.config['SQLALCHEMY_DATABASE_URI'] ='mysql://flask:admin@localhost/flaskdb'
app.config['SQLALCHEMY_COMMIT_ON_TEARDOWN'] = True
```
SQLALCHEMY_TRACK_MODIFICATIONS
如果设置成 True (默认情况)，Flask-SQLAlchemy 将会追踪对象的修改并且发送信号。
SQLALCHEMY_DATABASE_URI
连接数据库,这里连接mysql: 

> mysql://username:password@hostname/database

SQLALCHEMY_COMMIT_ON_TEARDOWN
将其设为 True时,每次请求结束后都会自动提交数据库中的变动。
# 定义模型
```
class User(db.Model):
    __tablename__ = 'users'
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(20), unique=True)

    def __repr__(self):
        return '<User %r>' % self.name
```
可以使用
+ ForeignKey() 构造外键
+ primary_key=True 定义主键
+ unique=True 唯一约束
+ index=True 创建索引
等等
# 数据库操作
```
(venv) $ python hello.py shell
>>> from hello import db
>>> db.create_all() 					#创建表
>>> db.drop_all() 					#删除表
>>> db.session.add(user_somebody) 			#数据添加到会话
>>> db.session.commit()				#提交会话
>>> db.session.rollback() 				#数据回滚
>>> db.session.delete(user_somebody)			#删除某行数据
>>> User.query.all() 					#查询所有行
>>> User.query.filter_by(name=somebody).all()		#条件查询
```
# 常用过滤器
> filter() 	把过滤器添加到原查询上,返回一个新查询
> filter_by() 	把等值过滤器添加到原查询上,返回一个新查询
> limit() 	使用指定的值限制原查询返回的结果数量,返回一个新查询
> offset() 	偏移原查询返回的结果,返回一个新查询
> order_by() 	根据指定条件对原查询结果进行排序,返回一个新查询
> group_by() 	根据指定条件对原查询结果进行分组,返回一个新查询

# 常用执行函数
> all() 		以列表形式返回查询的所有结果
> first() 		返回查询的第一个结果,如果没有结果,则返回 None
> first_or_404() 	返回查询的第一个结果,如果没有结果,则终止请求,返回 404 错误响应
> get() 		返回指定主键对应的行,如果没有对应的行,则返回 None
> get_or_404() 	返回指定主键对应的行,如果没找到指定的主键,则终止请求,返回 404 错误响应
> count() 	返回查询结果的数量
> paginate() 	返回一个 Paginate 对象,它包含指定范围内的结果
