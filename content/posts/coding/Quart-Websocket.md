---
title: "Quart Websocket"
date: "2025-05-02T16:00:00+08:00"
categories: ["代码","Python"]
tags: ["Python","Flask","Websocket"]
---

# Quart
可以简单理解为Flask的异步升级版本, 支持异步, 支持协程, 支持websocket
## Quart Websocket
对websocket做一层简单的封装

```python
from quart import Quart as Flask, websocket, Websocket
import quart as flask
import asyncio

app = Flask(__name__)


class Manager(object):
    map_lock = asyncio.Lock()
    conn_map: dict[Websocket, asyncio.Queue] = {}

    async def broadcast(self, data):
        async with self.map_lock:
            for queue in self.conn_map.values():
                try:
                    await queue.put(data)
                except Exception as e:
                    print(f"广播数据到队列时出错: {e}")

    async def join(self, key: Websocket):
        buf = asyncio.Queue()
        async with self.map_lock:
            self.conn_map[key] = buf

    async def leave(self, key: Websocket):
        async with self.map_lock:
            self.conn_map.pop(key, None)

    async def handle_recv(self, websocket: Websocket):
        try:
            while True:
                data = await websocket.receive()
                await self.broadcast(data)
        except asyncio.CancelledError:
            raise
        except BaseException as e:
            print(f"handle_recv err: {e}")

    async def handle_send(self, websocket: Websocket):
        queue = self.conn_map.get(websocket)
        try:
            while True:
                data = await queue.get()
                await websocket.send(data)
        except asyncio.CancelledError:
            raise
        except BaseException as e:
            print(f"handle_send 出错: {e}")


manager = Manager()


@app.websocket('/ws')
async def ws():
    key = websocket

    await manager.join(key)

    tasks = [
        asyncio.create_task(manager.handle_recv(key)),
        asyncio.create_task(manager.handle_send(key)),
    ]
    try:
        # 这里必须使用gather, 而不是wait
        await asyncio.gather(*tasks, )
        # asyncio.wait(tasks, return_when=asyncio)
    except asyncio.exceptions.CancelledError:
        print('disconnect')
    except BaseException as e:
        print(f"WebSocket 处理出错: {e}", type(e))
    finally:
        await manager.leave(key)


if __name__ == '__main__':
    app.run(debug=False)

```