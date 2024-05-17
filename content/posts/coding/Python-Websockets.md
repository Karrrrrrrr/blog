---
title: "Python 原生Websocket"
date: "2024-05-17T15:00:00+08:00"
categories: [ "代码","Python" ]
tags: [ "Python",Websocket" ]
---

# Python原生Websocket

```python
import asyncio
from websockets import (
    WebSocketServerProtocol,
    serve as serve_websocket,
    ConnectionClosedOK
)

mp = {}


async def handle_service(sock: WebSocketServerProtocol):
    ip = sock.remote_address[0]
    port = sock.remote_address[1]
    addr = f'{ip}:{port}'
    mp[addr] = sock
    try:
        while 1:
            data = await sock.recv()
            print(data)
            for e in mp:
                ws = mp[e]
                await ws.send(data)
    except ConnectionClosedOK as e:
        print('any one exit...', addr)
        mp.pop(addr)


async def handler(ws: WebSocketServerProtocol, path: str):
    await handle_service(ws)


if __name__ == '__main__':
    srv = serve_websocket(handler, "0.0.0.0", 5000)
    loop = asyncio.get_event_loop()
    loop.run_until_complete(srv)
    loop.run_forever()
```