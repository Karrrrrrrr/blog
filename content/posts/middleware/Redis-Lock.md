---
title: "redis 分布式锁"

date: 2024-06-28T15:30:00+08:00 


categories: ["代码", "分布式锁", "redis" ]

tags: ["redis", "分布式锁" ]
---

# Redis 分布式锁

## 安装

### docker

```shell
docker pull cgr.dev/chainguard/redis:latest
```

### docker-compose

```yaml
version: '3.7'
services:
  redis-lock:
    image: cgr.dev/chainguard/redis:latest
    container_name: redis-lock
    ports:
      - "6379:6379"
```
## redsync 源码实现

### v8
```go
package main

import (
	"fmt"
	redis "github.com/go-redis/redis/v8"
	redsync "github.com/go-redsync/redsync/v4"
	goredis "github.com/go-redsync/redsync/v4/redis/goredis/v8"
	"time"
)

func main() {
	client := redis.NewClient(&redis.Options{
		Addr: "192.168.80.128:6379",
	})
	pool := goredis.NewPool(client)
	rs := redsync.New(pool)
	mutexname := "lock:1"
	//创建基于key的互斥锁
	var fn = func() {
		mutex := rs.NewMutex(mutexname, redsync.WithTries(1000))

		// 对key进行
		if err := mutex.Lock(); err != nil {
			panic(err)
		}
		// 模拟业务耗时
		fmt.Println("ok")
		time.Sleep(1 * time.Second)
		if ok, err := mutex.Unlock(); !ok || err != nil {
			panic("unlock failed")
		}
	}
	for i := 0; i < 30; i++ {
        // 模拟并发
		go fn()
	}
	select {}
}

```

### v9

```go
package main

import (
	"fmt"
	"github.com/go-redsync/redsync/v4"
	"github.com/go-redsync/redsync/v4/redis/goredis/v9"
	redis "github.com/redis/go-redis/v9"
	"time"
)

func main() {
	client := redis.NewClient(&redis.Options{
		Addr: "192.168.80.128:6379",
	})
	pool := goredis.NewPool(client)
	rs := redsync.New(pool)
	mutexname := "lock:1"
	//创建基于key的互斥锁
	var fn = func() {
		mutex := rs.NewMutex(mutexname, redsync.WithTries(1000))

		// 对key进行
		if err := mutex.Lock(); err != nil {
			panic(err)
		}

		// 获取锁后的业务逻辑处理.
		fmt.Println("ok")
		time.Sleep(1 * time.Second)
		// 释放互斥锁
		if ok, err := mutex.Unlock(); !ok || err != nil {
			panic("unlock failed")
		}
	}
	for i := 0; i < 30; i++ {
		go fn()
	}
	select {}
}

```

## 注意点

初始化mutex的时候, 默认参数是重试32次,锁的超时时间是8秒,如果不满足业务需求,需要手动配置

关于延迟函数为什么要加入随机因子? 因为需要实现公平锁,如果没有随机,那么永远是先来的先抢到锁

```go
package redsync
// NewMutex returns a new distributed mutex with given name.
func (r *Redsync) NewMutex(name string, options ...Option) *Mutex {
    m := &Mutex{
        name:   name,
        expiry: 8 * time.Second,
        tries:  32,
        delayFunc: func (tries int) time.Duration {
            return time.Duration(rand.Intn(maxRetryDelayMilliSec-minRetryDelayMilliSec)+minRetryDelayMilliSec) * time.Millisecond
            },
        genValueFunc:  genValue,
        driftFactor:   0.01,
        timeoutFactor: 0.05,
        quorum:        len(r.pools)/2 + 1,
        pools:         r.pools,
    }
    for _, o := range options {
        o.Apply(m)
    }
    if m.shuffle {
        randomPools(m.pools)
    }
    return m
}
```