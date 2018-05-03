---
title: mongodb
date: 2018-04-27 16:11:00
tags: [python, mongodb]
---

# 安装

[windows download](https://www.mongodb.com/download-center?jmp=tutorials&_ga=2.119332430.697798098.1524811150-1838574771.1524811150#community)

[windwos install](https://docs.mongodb.com/manual/tutorial/install-mongodb-on-windows/)

mongod.exe所在目录`C:\Program Files\MongoDB\Server\3.6\bin\mongod.exe`

MongoDB需要建立一个目录用来存放所有数据，默认为`\data\db`

这里自己建立一个目录
```
md d:\mongodb\db
```

指定目录启动
```
mongod --dbpath d:\mongodb\db
```

或者通过配置文件启动

[配置参数](https://docs.mongodb.com/v2.4/reference/configuration-options/)

建立一个`mongod.conf`
```
dbpath = d:\mongodb\db
```

利用配置文件启动
```
mongod -f d:\mongodb\mongod.conf
```

## 配置Windows Server

创建log文件夹：
```
mkdir d:\mongodb\log
```

建立MongoDB配置文件 `C:\Program Files\MongoDB\Server\3.6\mongod.cfg`
```
systemLog:
    destination: file
    path: c:\data\log\mongod.log
storage:
    dbPath: c:\data\db
```

安装MongoDB 服务
```
mongod --config "C:\Program Files\MongoDB\Server\3.6\mongod.cfg" --install
```

启动MongoDB 服务
```
net start MongoDB
```

关闭MongoDB 服务
```
net stop MongoDB
```

删除MongoDB 服务
```
mongod --remove
```

# 权限配置

使用`mongo`登录数据库
```
mongo
```

首先添加一个具有`userAdminAnyDatabase`权限的用户，绑定`admin`库，这个用户只负责管理其他用户，不对数据库进行操作。
```
> use admin
> db.createUser({ user: "useradmin", pwd: "123456", roles: [{ role: "userAdminAnyDatabase", db: "admin" }] })
```

使用这个账户登录
```
mongo -u "useradmin" -p "123456" --authenticationDatabase "admin"
```

针对test库添加一个satncs用户
```
> use test
> db.createUser({user: "satncs", pwd: "satncs", roles: [{role: "dbOwner", db: "test"}]})
```

这样用户satncs就添加成功了，如果mongodb compass 连接显示报错，重启一下客户端就行。

# 连接

URL连接格式
```
mongodb://[username:password@]host1[:port1][,host2[:port2],...[,hostN[:portN]]][/[database][?options]]
```
