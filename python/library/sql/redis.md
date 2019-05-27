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

# Pub/Sub模式
aioredis提供Redis的消息订阅功能。

客户端可以订阅频道和发布消息，由客户端发布的消息会被转发给订阅该频道的所有其他客户端。

订阅频道如下
```
import asyncio
import aioredis

async def sub_read_loop(channel):
    while await channel.wait_message():
        msg = await channel.get()
        print(msg)

async def create_sub_cli():
    sub_cli = await aioredis.create_redis(
        'redis://{}:{}/{}'.format(
            '192.168.0.201',
            6379,
            1
        )
    )
    channel, *_ = await sub_cli.subscribe('channel_001', 'channel_002')
    asyncio.create_task(sub_read_loop(channel))
```
这里订阅了频道`channel_001`和`channel_002`

当一个客户端订阅一个频道之后，Redis不会接收其他任何命令，例如如下操作会报错
```
async def create_sub_cli():
    sub_cli = await aioredis.create_redis(
        'redis://{}:{}/{}'.format(
            '192.168.0.201',
            6379,
            1
        )
    )
    channel, *_ = await sub_cli.subscribe('channel_001', 'channel_002')
    asyncio.create_task(sub_read_loop(channel))

    await sub_cli.set('b', 1)
    res = await sub_cli.get('b')
    print(res)
```

循环发布消息
```
async def pub_msg_loop(sub_cli, channel):
    while True:
        await asyncio.sleep(1)
        await sub_cli.publish(channel, b'hello')

async def create_pub_cli():
    pub_cli = await aioredis.create_redis(
        'redis://{}:{}/{}'.format(
            '192.168.0.201',
            6379,
            1
        )
    )
    asyncio.create_task(pub_msg_loop(pub_cli, 'channel_001'))
    asyncio.create_task(pub_msg_loop(pub_cli, 'channel_002'))
```
这里会循环向频道`channel_001`和`channel_002`发送消息

和订阅不同，发布消息的client可以使用其他命令
```
async def create_pub_cli():
    pub_cli = await aioredis.create_redis(
        'redis://{}:{}/{}'.format(
            '192.168.0.201',
            6379,
            1
        )
    )
    asyncio.create_task(pub_msg_loop(pub_cli, 'channel_001'))
    asyncio.create_task(pub_msg_loop(pub_cli, 'channel_002'))

    await pub_cli.set('a', 1)
    res = await pub_cli.get('a')
    print(res)
```

启动测试
```
async def main():
    await create_sub_cli()
    await create_pub_cli()
    await asyncio.sleep(10000)


if __name__ == '__main__':
    asyncio.run(main())
```

打印
```
b'hello'
b'hello'
b'hello'
...
```
