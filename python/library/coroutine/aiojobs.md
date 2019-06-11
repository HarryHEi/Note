---
title: aiojobs
date: 2019-06-11 15:17:00
tags: [python]
---

# aiojobs
[aiojobs](https://github.com/aio-libs/aiojobs)用于管理后台任务，相比较直接使用task更加方便。

# 使用场景
例如在一个客户服务程序的使用场景下，需要针对每个用户开启一个处理循环，比如循环写和循环读。
```
class Server:
    @staticmethod
    async def _receive_loop(client):
        while True:
            print('Receive from client {}...'.format(client))
            await asyncio.sleep(2)

    @staticmethod
    async def _write_loop(client):
        while True:
            print('Write to client {}...'.format(client))
            await asyncio.sleep(2)
```
每个用户可能有多个处理循环，用户中途可能登录和登出，需要在适当时候开启和关闭这些循环，使用aiojobs可以方便地管理这些后台任务。
```
class Server:
    def __init__(self):
        self.schedulers = {}

    async def add_client(self, client):
        if client not in self.schedulers:
            print('Client {} connected.'.format(client))
            scheduler = await aiojobs.create_scheduler()
            self.schedulers[client] = scheduler
            await scheduler.spawn(self._receive_loop(client))
            await scheduler.spawn(self._write_loop(client))

    async def remove_client(self, client):
        if client in self.schedulers:
            print('Client {} disconnected.'.format(client))
            scheduler: aiojobs.Scheduler = self.schedulers.pop(client)
            await scheduler.close()
```
`scheduler`通过`spawn`开启任务，`cancel`可以取消所有任务。

测试

```
async def test_loop():
    server = Server()
    await server.add_client('001')
    await server.add_client('002')
    await server.add_client('003')

    await asyncio.sleep(5)
    await server.remove_client('002')

    await asyncio.sleep(5)
    await server.remove_client('003')

    await asyncio.sleep(5)
    await server.remove_client('001')

    await asyncio.sleep(5)


if __name__ == '__main__':
    asyncio.run(test_loop())
```
