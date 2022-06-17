---
title: "Flask Websocket问题"
date: 2022-06-17T23:44:35+08:00 categories: ["代码","Python" ]
tags: ["Python","Flask" , "Websocket" ]
---

## Flask v2

### 问题的起因

flask更新了v2之后， 原先的 Flask + gevent-websocket 无法正常工作， 具体体现在Chrome下使用Websocket连接上马上断开（staus从1变成3） 其他chrome内核浏览器均有该问题，
firefox正常 以下是问题代码

```python
from flask import Flask,request,render_template
from  geventwebsocket.websocket import WebSocket,WebSocketError
from  geventwebsocket.handler import WebSocketHandler
from  gevent.pywsgi import WSGIServer

import json

app = Flask(__name__)
user_socket_dict={}

@app.route('/ws')
def ws(username):
    user_socket=request.environ.get("wsgi.websocket")
    if not user_socket:
        return "请以WEBSOCKET方式连接"

    while True:
        try:
            user_msg = user_socket.receive()
            for user_name,u_socket in user_socket_dict.items():

                who_send_msg={
                    "send_user":username,
                    "send_msg":user_msg
                }

                if user_socket == u_socket:
                    continue
                u_socket.send(json.dumps(who_send_msg))

        except WebSocketError as e:
            user_socket_dict.pop(username)

if __name__ == '__main__':
    http_serve=WSGIServer(("0.0.0.0",5000),app,handler_class=WebSocketHandler)
    http_serve.serve_forever()
```

### 推测

具体是什么原因不得而知， 但是从chrome能一瞬间的连接成功， 可以推测出Flask能成功收到ws连接， 但是没有进入handler。 所以大胆的假设ws能进入before_request hook, 经过验证， 推测是成立的，
Websocket只存在于before_request。

### 解决方案

抽离出websocket处理函数， 在before_request hook中调用处理函数， 并且过滤普通请求

```python
from flask import  Flask ,request,render_template
from  geventwebsocket.websocket import WebSocket,WebSocketError
from  geventwebsocket.handler import WebSocketHandler
from  gevent.pywsgi import WSGIServer

import json

app = Flask(__name__)
# user_socket_dict={}
user_socket_set = set()

def websocket_handler(user_socket):
    s.add(user_socket)
    while True:
        try:
            user_msg = user_socket.receive()
            for u_socket in user_socket_dict.items():

              
                if user_socket == u_socket:
                    continue
                u_socket.send(user_msg)

        except WebSocketError as e:
            # user_socket_dict.pop(username)
            s.remove(user_socket)



@app.before_request
def before():
    user_socket=request.environ.get("wsgi.websocket")
    if user_socket:
        websocket_handler(user_socket)
        return

    
if __name__ == '__main__':
    http_serve=WSGIServer(("0.0.0.0",5000),app,handler_class=WebSocketHandler)
    http_serve.serve_forever()
```

