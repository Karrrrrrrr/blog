---
title: "etcd 分布式锁"

date: 2024-06-28T14:00:00+08:00 


categories: ["代码", "分布式锁", "etcd" ]

tags: ["etcd", "分布式锁" ]
---
# etcd和分布式锁

## 安装

```cgr.dev/chainguard```
是国内下载镜像比较快的地址,一般只有最新版,类似滚动更新,这里必须配置```ETCD_LISTEN_CLIENT_URLS=http://0.0.0.0:2379```
允许宿主机访问

如果是 ```bitnami/etcd```的镜像, 则不需要手动配置```ETCD_LISTEN_CLIENT_URLS```
### docker-compose 
````yaml    
version: '3.7'
services:
  etcd:
    image: cgr.dev/chainguard/etcd:latest
    #    image: "bitnami/etcd"
    container_name: etcd
    environment:
      - ALLOW_NONE_AUTHENTICATION=yes
      # 配置公网访问
      - ETCD_LISTEN_CLIENT_URLS=http://0.0.0.0:2379
    ports:
      - "2380:2380"
      - "2379:2379"
````

### 分布式锁实现
注意,这里需要用```go.etcd.io/etcd/client/v3```这个包

网上说的```github.com/coreos/etcd/clientv3```这个包,因为和grpc版本不匹配已经不能用了

```go
package main

import (
	"fmt"
	etcd "go.etcd.io/etcd/client/v3"
	"go.etcd.io/etcd/client/v3/concurrency"
	"time"
)

var (
	cli *etcd.Client
)

func Service() error {
	session, err := concurrency.NewSession(cli)
	if err != nil {
		return err
	}

	locker := concurrency.NewLocker(session, "Id:1")
	locker.Lock()
	defer locker.Unlock()

	// do any thing...
	// 模拟耗时任务
	time.Sleep(time.Second)
	fmt.Println("?")
	return nil
}
func main() {
	var err error
	cli, err = etcd.New(etcd.Config{
		Endpoints:   []string{"192.168.80.128:2379"},
		DialTimeout: 5 * time.Second,
	})

	if err != nil {
		panic(err)
	}
	var ch = make(chan int, 1)
	for i := 0; i < 30; i++ {
		go Service()
	}
	<-ch

}
```