---
title: sqlalchemy
date: 2018-04-27 13:50:00
tags: [python]
---

通常在连接数据库时会使用相应数据库的Python库，比如连接sqlite使用sqlite3，连接mysql使用pymysql。一般会提供一个`execute()`接口用于使用数据库语句。

在构造语句时通常使用`?`作为占位符，比如：
```
c.execute('update msgs set state = ? where uid="1"', ('sent', ))
```

sqlalchemy提供应用级别的数据库操作，并且支持ORM（对象关系映射）。

[sqlalchemy core](http://docs.sqlalchemy.org/en/latest/core/tutorial.html)

sqlalchemy连接sqlite用法
```
from sqlalchemy import create_engine
engine = create_engine('sqlite:///:memory')
```

可以使用(os.path.join)构造路径并打开连接。
```
file_path = os.path.join(os.path.dirname(os.path.realpath(__file__)), 'test.db')
create_engine('sqlite:///{}'.format(file_path), echo=True)
```

对于mysql，连接时指定用户名以及密码，以及pool_recycle时间
```
engine = create_engine(
    'mysql+pymysql://user:passwd@192.168.0.196/test?charset=utf8mb4',
    pool_recycle=3600,
    echo=True
)
```

在构建表对象之前，需要创建一个MetaData()的实例。
```
from sqlalchemy import MetaData
metadata = MetaData()
```

针对数据库中的表构建Table对象，比如构建一个`tests`表的对象
```
tests = Table(
    'tests',
    metadata,
    Column('id', Integer, primary_key=True),
    mysql_engine='InnoDB',
    mysql_charset='utf8mb4'
)
```

创建表，如果表不存在，会在数据库中创建
```
metadata.create_all(engine)
```

获取获取一个connect对象用于数据库操作
```
conn = engine.connect()
```

查询操作
```
from sqlchemy.sql import select
res = conn.execute(
	select([tests]).where(
		(tests.c.id > 1) && (tests.c.id < 10)
	)
)
```

插入操作
```
conn.execute(
	tests.insert(),
	id=1
)
```

删除操作
```
conn.execute(
	tests.delete().where(
		(tests.c.id > 1) && (tests.c.id < 10)
	)
)
```

sqlalchemy ORM

[sqlalchemy orm](http://docs.sqlalchemy.org/en/latest/orm/tutorial.html)

一对多关系

```
import sqlalchemy.ext.declarative
import sqlalchemy.orm
import os

database_file_path = os.path.join(os.path.dirname(os.path.realpath(__file__)), 'test.db')
engine = sqlalchemy.create_engine("sqlite:///{}".format(database_file_path), echo=True)

Base = sqlalchemy.ext.declarative.declarative_base()


class User(Base):
    __tablename__ = 'users'

    id = sqlalchemy.Column(sqlalchemy.Integer, sqlalchemy.Sequence('users_id_seq'), primary_key=True)
    name = sqlalchemy.Column(sqlalchemy.String(1024))
    addresses = sqlalchemy.orm.relation('Address', back_populates='user')

    def __repr__(self):
        return '<User(id={}, name={})>'.format(self.id, self.name)


class Address(Base):
    __tablename__ = 'addresses'

    id = sqlalchemy.Column(sqlalchemy.Integer, sqlalchemy.Sequence('addresses_id_seq'), primary_key=True)
    email_address = sqlalchemy.Column(sqlalchemy.String)
    user_id = sqlalchemy.Column(sqlalchemy.Integer, sqlalchemy.ForeignKey('users.id'))

    user = sqlalchemy.orm.relation('User', back_populates='addresses')

    def __repr__(self):
        return '<Address(id={}, email_address={})>'.format(self.id, self.email_address)

Base.metadata.create_all(engine)

Session = sqlalchemy.orm.sessionmaker()
Session.configure(bind=engine)
session = Session()

new_user = User(name='11112222')
new_user.addresses = [
    Address(email_address='1111@sina.com'),
    Address(email_address='2222@sina.com')
]

session.add(new_user)
session.commit()
```

多对多关系，需要有第三个表来保存关系

```
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import relation, sessionmaker
from sqlalchemy import create_engine, Integer, String, Sequence, Column, ForeignKey, Table
import os

database_file_path = os.path.join(os.path.dirname(os.path.realpath(__file__)), 'test.db')
engine = create_engine('sqlite:///{}'.format(database_file_path), echo=True)

Base = declarative_base()


post_keywords = Table(
    'post_keywords',
    Base.metadata,
    Column('post_id', ForeignKey('posts.id'), primary_key=True),
    Column('keyword_id', ForeignKey('keywords.id'), primary_key=True)
)

class Post(Base):
    __tablename__ = 'posts'

    id = Column(Integer, primary_key=True)
    body = Column(String(1024))

    keywords = relation('Keyword', secondary=post_keywords, back_populates='posts')

    def __init__(self, body):
        self.body = body

    def __repr__(self):
        return '<Post(id={}, body={})>'.format(self.id, self.body)


class Keyword(Base):
    __tablename__ = 'keywords'

    id = Column(Integer, primary_key=True)
    number = Column(Integer)

    posts = relation('Post', secondary=post_keywords, back_populates='keywords')

    def __init__(self, number):
        self.number = number

    def __repr__(self):
        return '<Keyword(id={}, number={})>'.format(self.id, self.number)


Base.metadata.create_all(engine)

Session = sessionmaker()
Session.configure(bind=engine)
session = Session()

new_post = Post(body='123')
new_post_2 = Post(body='111')

session.add_all([new_post, new_post_2])
session.commit()
```
