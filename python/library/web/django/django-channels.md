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
