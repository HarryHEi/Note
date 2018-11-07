---
title: mqtt
date: 2017-03-23 09:18:19
tags: [python]
---

# 简介

MQTT（Message Queuing Telemetry Transport，消息队列遥测传输）是基于TCP/IP的传输协议

# 在windows环境搭建MQTT服务器

参考：[链接](http://blog.csdn.net/yannanxiu/article/details/52703946)

注意：

- 要设置环境变量`APOLLO_HOME`，比如，本机设置的是`D:\App\mqtt\apache-apollo-1.7.1`

- 还有java环境变量`JAVA_HOME`，这里是`D:\App\java8\java\`


# 一个基于mqtt的聊天工具

注意：
	如果在连接服务器后不添加延时，直接上传，会被服务器误认为是攻击，触发一个`Errno 10054`异常，

一个MQTT客户端类

- `on_connect`会在连接到服务器后调用
	- `client`是传回的client参数
	- `user_data`是`client`实例化或者调用`userdata_set()`设置的`_userdata`参数
	- `flags`是`broker`返回的字典参数
	- `rc`连接结果，0表示成功
		- 1: Connection refused - incorrect protocol version
	    - 2: Connection refused - invalid client identifier
	    - 3: Connection refused - server unavailable
	    - 4: Connection refused - bad username or password
	    - 5: Connection refused - not authorised
- `on_message`在收到消息后调用，消息放在`msg`字符串中

```
import paho.mqtt.client as mqtt
import time


class MQTTClient(object):
    """一个MQTT客户端"""
    def __init__(self):
        self.client = mqtt.Client()
        self.client.username_pw_set("admin", "password")
        self.client.on_connect = self.on_connect
        self.client.on_message = self.on_message
        self.client.connect("127.0.0.1", 61613, 60)
        self.client.loop_start()

    def on_connect(self, client, user_data, flags, rc):
        del client, user_data, flags, rc
        self.client.subscribe("chat")

    @staticmethod
    def on_message(client, user_data, msg):
        del client, user_data
        print(msg.topic + " " + str(msg.payload))
```

实例化两个客户端互发

```
mc1 = MQTTClient()
mc2 = MQTTClient()
while 1:
    time.sleep(5)
    mc1.client.publish("chat", "hi1")
    time.sleep(5)
    mc2.client.publish("chat", "hi2")
```
