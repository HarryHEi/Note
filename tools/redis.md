---
title: redis
date: 2018-5-31 15:36:00
tags: [redis]
---

[redis常用指令](https://redis.io/commands)
[aioredis](http://aioredis.readthedocs.io/en/v1.1.0/index.html)


# python连接redis测试

```
import aioredis
import asyncio


@asyncio.coroutine
async def go():
    pool = await aioredis.create_redis_pool(
        'redis://192.168.99.100:32780',
        minsize=5,
        maxsize=10
    )
    # value
    await pool.set('my_number', 1)
    val = await pool.get('my_number')
    await pool.delete('my_number')
    print(val)

    # list
    await pool.lpush('my_list', 1)
    await pool.rpush('my_list', 2)
    val = await pool.lrange('my_list', 0, -1)
    await pool.delete('my_list')
    print(val)

    # set
    await pool.zadd('my_set', 4, 'b', 3, 'd')
    value = await pool.zrange('my_set', withscores=True)
    print(value)
    print(dict(value))
    await pool.delete('my_set')

    # tuple
    await pool.hmset('my_tuple', 'a', 'b', 'c', 'd')
    cur, value = await pool.hscan('my_tuple')
    print(cur, value)
    print(dict(value))
    await pool.delete('my_tuple')


loop = asyncio.get_event_loop()
loop.run_until_complete(go())
```