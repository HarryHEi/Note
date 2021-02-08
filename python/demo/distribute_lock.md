---
title: 基于Redis的分布式锁
date: 2020-09-22 18:20:33
tags: [python]
---

## 原理

通过Redis实现锁有这么几点：

+ Redis单线执行来自客户端的语句。
+ 多个语句可以通过lua脚本保证原子性。
+ 通过expire实现锁超时。
+ 过高的并发会导致锁性能变差。

流程如下：

+ 生成唯一标识。
+ 设置值（key为锁名称、仅在没有该key时返回成功、设置超时时间）
+ 设置值失败时继续尝试设置值，直到成功或达到重试次数上限。
+ 值设置成功则成功获得锁。
+ 执行业务操作。
+ 获取值（key为锁名称），比较值是否与之前的唯一标识相同，相同则删除锁，否则回滚业务操作。

## 实现

首先需要实现一个类表示锁。

包含两个参数：锁名称和锁值。

```
import dataclasses

@dataclasses.dataclass()
class Lock:
    key: str
    val: str
```

随后是锁的实现，先创建一个类，初始化参数有三个：

+ 第一个是redis的地址，这个是必填的。
+ 后面两个参数分别表示重试次数和重试延迟时间

```
import redis

class RedisLockManager:
    default_retry_count = 3
    default_retry_delay = 0.2
    unlock_script = """
    if redis.call("get",KEYS[1]) == ARGV[1] then
        return redis.call("del",KEYS[1])
    else
        return 0
    end"""
    
    def __init__(self, redis_url, retry_count=None, retry_delay=None):
        self.server = redis.StrictRedis.from_url(redis_url)
        self.retry_count = retry_count or self.default_retry_count
        self.retry_delay = retry_delay or self.default_retry_delay
```

获得锁和释放锁分别对应redis中值得设置和删除，设置锁需要用到`nx`参数，该参数设置为`True`时表示只有不含有该key时才能设置成功，`px`则表示超时时间，单位是毫秒。redis官方代码的注释可以看出来。

```
``px`` sets an expire flag on key ``name`` for ``px`` milliseconds.

``nx`` if set to True, set the value at key ``name`` to ``value`` only
if it does not exist.
```

实现设置值的方法

```
class RedisLockManager:
    ...
    def lock_instance(self, key, val, ttl):
        return self.server.set(key, val, nx=True, px=ttl)
```

有了设置值的方法后，可以很方便的实现获得锁的逻辑。

```
class RedisLockManager:
    ...
    def get_lock(self, key, ttl):
        retry = 0
        val = uuid.uuid1().hex

        while retry < self.retry_count:
            locked = self.lock_instance(key, val, ttl)
            if locked:
                return Lock(key, val)
            else:
                retry += 1
                time.sleep(self.retry_delay)
        return False
```

释放锁的逻辑是先通过key获取值，再比较当前的值跟之前获得锁生成的值是否相同，只有在相同的情况下才会删除锁，返回删除key的个数，这里是1，否则返回0。

为了保证原子性，也就是在查看key值和删除key之前不能有其他客户端读写操作，需要通过lua脚本实现。

```
class RedisLockManager:
    ...
    unlock_script = """
    if redis.call("get",KEYS[1]) == ARGV[1] then
        return redis.call("del",KEYS[1])
    else
        return 0
    end"""
    
    def unlock_instance(self, key, val):
        return self.server.eval(self.unlock_script, 1, key, val)
```

释放锁的方法也很简单。

```
class RedisLockManager:
    ...
    def unlock(self, lock):
        return self.unlock_instance(lock.key, lock.val)
```

使用方式。

```
>>> rml = RedisLockManager('redis://172.172.177.191:6379/1')
>>> lock = rml.get_lock('my_lock_1', 10000)
>>> rml.unlock(lock)
1
```

