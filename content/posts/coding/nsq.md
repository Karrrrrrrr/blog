
---
title: "Python NSQ"
date: 2024-08-10T00:19:18+08:00
categories: ["代码","NSQ" ]
tags: ["Python","消息队列" ,"NSQ"]

---

# Python NSQ

官方的nsq库强依赖于asyncio和tornado,不好用,建议使用第三方的客户端 (nsq-py/gnsq)
## 安装
```shell
pip install nsq-py
```

## 消费
```python
# Windows 下使用
from nsq.reader import Reader
from nsq.client import Client
# Linux 使用
# from nsq.gevent import Reader,Client
# 必须是byte类型
reader = Reader(b'topic', b'channel', nsqd_tcp_addresses=['localhost:4150'])

for message in reader:
    print('message:', message)
    # 必须调用fin表示消费成功
    message.fin()


```


```python
# Windows 下使用
from nsq.reader import Reader
from nsq.client import Client
# Linux 使用
# from nsq.gevent import Reader,Client


cli = Client(nsqd_tcp_addresses=['192.168.80.128:4150'])

# 必须是byte类型
cli.pub(b'topic', b'test')

```