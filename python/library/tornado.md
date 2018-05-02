---
title: tornado
date: 2018-05-2 16:03:00
tags: [python]
---

tornado是一个异步web框架，并且也提供http服务。

定义一个`RequestHandler`，tornadot提供简单的web异步框架
```
class MainHandler(tornado.web.RequestHandler):
    def get(self):
        self.write("Hello, world")
```
构建`Application`
```
tornado.web.Application([
    (r"/", MainHandler),
])
```
启动`HTTPServer`，进行事件循环。
```
server = HTTPServer(app)
server.listen(8888)
IOLoop.current().start()
```

使用`@tornado.web.gen.coroutine`自定义并发执行Loop
```
def do_something():
    print('something', time.time())


def do_another():
    print('another', time.time())


@tornado.web.gen.coroutine
def test_loop():
    while True:
        yield do_something()
        yield tornado.web.gen.sleep(1)


@tornado.web.gen.coroutine
def another_loop():
    while True:
        yield do_another()
        yield tornado.web.gen.sleep(2)
```
在启动时，先`spawn_callback`自定义的Loop，然后调用`start()`阻塞。
```
tornado.ioloop.IOLoop.current().spawn_callback(another_loop)
tornado.ioloop.IOLoop.current().spawn_callback(test_loop)
tornado.ioloop.IOLoop.current().start()
```

tornado提供`tornado.wsgi.WSGIAdapter`将一个`Application`转为WSGI接口的应用，然后使用其他支持WSGI的服务启动，比如使用`gevent.pywsgi.WSGIServer`
```
wsgi_app = tornado.wsgi.WSGIAdapter(app)
server = gevent.pywsgi.WSGIServer(('192.168.0.114', 7776), wsgi_app)
server.serve_forever()
```

如果转化为WSGI，那么异步方法将不再支持，比如`AsyncHTTPClient`、`tornado.auth`以及`tornado.websocket`。

如果希望支持`websocket`，可以使用其他库，如果`ws4py`
```
from ws4py.websocket import WebSocket
import ws4py.server.geventserver
from ws4py.server.wsgiutils import WebSocketWSGIApplication

ws_handlers = []

class TestWebSocket(WebSocket):
    def opened(self):
        print('connected')
        ws_handlers.append(self)

    def closed(self, code, reason=None):
        print('closed {}, {}'.format(code, reason))
        ws_handlers.remove(self)

    def received_message(self, message):
        self.send(message.data, message.is_binary)


ws_app = WebSocketWSGIApplication(handler_cls=TestWebSocket)
ws_server = ws4py.server.geventserver.WSGIServer(('192.168.0.114', 7777), ws_app)
ws_server.serve_forever()
```

tornado提供TCPServer
```
import tornado.web
import tornado.ioloop
import tornado.queues
import tornado.gen
import tornado.tcpserver
import tornado.httpserver
import os
import tornado.auth


q = tornado.queues.Queue()


class TCPChannel(tornado.tcpserver.TCPServer):
    @tornado.gen.coroutine
    def handle_stream(self, stream, address):
        while True:
            data = yield stream.read_bytes(20)
            yield q.put(data)


@tornado.gen.coroutine
def handle_loop():
    while True:
        data = yield q.get()
        print('handle data ', data)
        yield tornado.gen.sleep(1)


class MainHandler(tornado.web.RequestHandler):
    def get(self, *args, **kwargs):
        self.redirect('/static/template/index.html')

    def post(self, *args, **kwargs):
        print(self.request.files)
        self.redirect('/')
        for index, file in self.request.files:
            with open('index' + '.pem', 'w') as f:
                f.write(file.body)


server = TCPChannel()
server.listen(8888)
app = tornado.web.Application(
    handlers=[
        (r'/', MainHandler),
        (r'/static/*', StaticHandle),
        (r'/random', APIHandler),
        (r'/json', JSONAPIHandler)
    ],
    **setting
)
app.listen(9999)
tornado.ioloop.IOLoop.current().spawn_callback(handle_loop)
tornado.ioloop.IOLoop.current().start()
```
