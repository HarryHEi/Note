---
title: peewee
date: 2018-11-7 10:29:00
tags: [python, sql]
---

# peewee
[peewee](http://docs.peewee-orm.com/en/latest/)是个轻量级的ORM框架。

# peewee_async
[peewee_async](https://github.com/05bit/peewee-async)是peewee的协程版本。

## 连接
```
db = peewee_async.MySQLDatabase(
    host='localhost',
    user='satncs',
    password='satncs',
    database='ncp_03_model',
    charset='utf8mb4',
)
```

## 定义model
定义一个Model基类，存放db实例。
```
class BaseModel(peewee.Model):
    class Meta:
        database = db
```
继承基类，定义Model
```
class User(BaseModel):
    name = peewee.CharField(unique=True)
```

## 创建表
自动创建表
```
db.create_tables([
    User,
])
```

## 创建全局manager
```
db.set_allow_sync(False)  # 不允许同步操作。
dbc_objects = peewee_async.Manager(db)
```

## N + 1 问题
使用join方法关联查询，避免产生额外的查询次数。

例如查询用户，包含用户所属的部门名称。
```
users = User.select(User, Dept.name).join(Dept)
```
反过来，查询部门时也可以预读用户。
```
dept_query = Dept.select()
user_query = User.select()
depts = await dbc_objects.prefetch(dept_query, user_query)
```
预读多张表
```
company_query = Company.select()
dept_query = Dept.select()
user_query = User.select()
depts = await dbc_objects.prefetch(company_query, dept_query, user_query)
```

## 分页
paginate(page, page_size)方法用户分页查询。
```
page = 2
page_size = 10
# 查询的内容
query = Log.select()
# 分页查询
pagination_query = query.paginate(page, page_size)
```
如果希望按照`count`、`next_page`、`result`方式返回，可以先查询以下数量，然后计算下一页页码。
```
page = 2
page_size = 10
# 查询的内容
query = Log.select()
# 拷贝一个query实例用于计算数量
query_copy = copy.copy(query)
# 计算数量
count = awat dbc_objects.count(query_copy)
# 根据总数、page_size和page计算下一页页码
if count > page * page_size:
    next_page = page + 1
else:
    next_page = None
# 分页查询
pagination_query = query.paginate(page, page_size)
# 查询结果
result = await dbc_objects.execute(pagination_query)
return {
    'count': count,
    'next_page': next_page,
    'result': res
}
```
