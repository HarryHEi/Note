---
title: Django Channels
date: 2019-06-22 14:29:00
tags: [django]
---

# ASGI
[ASGI](https://asgi.readthedocs.io/en/latest/)（异步服务网关接口）是WSGI的精神继承者，旨在在支持异步服务的Python Web服务器、框架和应用程序之间提供标准接口。

WSGI提供同步标准接口，ASGI同时为同步和异步应用提供标准接口。

# Channels
[Channels](https://channels.readthedocs.io/en/latest/introduction.html)使得Django能够编写异步代码以不仅仅支持HTTP，还可以支持需要长连接的WebSocket、MQTT等。

每个consumer都是一个有效的ASGI应用程序。

## 安装和配置
### 安装依赖
```
pip install channels
```

添加到`INSTALLED_APPS`
```
INSTALLED_APPS = [
    # ...
    'channels',
]

```

### 创建入口文件
`dj_demo/dj_demo/routing.py`

```
from django.conf.urls import url
from channels.routing import ProtocolTypeRouter
from channels.routing import URLRouter
from channels.http import AsgiHandler


application = ProtocolTypeRouter({
    'http': URLRouter([
        url(r'', AsgiHandler)
    ]),
})
```
`http`中的`URLRouter`以`url(r'', AsgiHandler)`作为最后一个路由，代理原本的WSGI请求。

在`settings.py`中指定入口文件
```
ASGI_APPLICATION = "dj_demo.routing.application"
```

### 配置`Channel Layer`
[Channel Layer](https://channels.readthedocs.io/en/latest/topics/channel_layers.html)

配置`Redis Chanel Layer`用作`Consumer`间通信。

安装依赖
```
pip install channels_redis
```
在`settings.py`中配置`Channel Layer`
```
CHANNEL_LAYERS = {
    'default': {
        'BACKEND': 'channels_redis.core.RedisChannelLayer',
        'CONFIG': {
            'hosts': [
                ('127.0.0.1', 6379),
            ],
        }
    }
}
```

### WebSocket
`WebSocket`的[ASGI接口](https://asgi.readthedocs.io/en/latest/specs/www.html#websocket)。

在Django某个APP中创建一个支持`WebSocket`的`Consumer`，文件`demo.consumers.py`
```
class EchoConsumer(AsyncWebsocketConsumer):
    async def connect(self):
        await self.accept()

    async def receive(self, text_data=None, bytes_data=None):
        await self.send(text_data)
```
这是一个简单的`Echo Websocket`服务。

在`routing.py`中指定路由。
```
from django.conf.urls import url
from channels.routing import ProtocolTypeRouter
from channels.routing import URLRouter
from channels.http import AsgiHandler

from demo.consumers import EchoConsumer

application = ProtocolTypeRouter({
    'http': URLRouter([
        url(r'', AsgiHandler)
    ]),
    'websocket': URLRouter([
        url(r'ws/', EchoConsumer),
    ])
})
```

### 部署
[部署](https://channels.readthedocs.io/en/latest/deploying.html)方式略有不同，这里使用[daphne](https://github.com/django/daphne)部署ASGI服务。

创建`dj_demo/dj_demo/asgi.py`入口文件

```
import os

import django
from channels.routing import get_default_application


os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'dj_demo.settings')
django.setup()
application = get_default_application()
```

利用`daphne`和`supervisor`启动多个服务进程，暴露本地的一个端口，`supervisor`配置如下

```
[fcgi-program:dj_demo]
socket=tcp://localhost:10234
command=/dj_demo/venv/bin/daphne -u /dj_demo/dj_demo%(process_num)d.sock --fd 0 --access-log - --proxy-headers dj_demo.asgi:application
directory=/dj_demo
numprocs=4
process_name=dhgraph%(process_num)d
user=harry
autostart=true
autorestart=false
stdout_logfile=/dj_demo/logs/dj_demo.log
redirect_stderr=true
```
使用`sudo service supervisor start`启动服务。
