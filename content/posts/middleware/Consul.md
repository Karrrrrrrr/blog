---
date: 2022-05-20T00:53:09+08:00

title: Consul的使用
cover: https://pic3.zhimg.com/80/v2-b676a9e59b4a277bfb4459b90e945fd2_720w.jpg
categories: ["代码" , "微服务" ,"中间件"]
tags: ["代码" , "微服务" ,"中间件"]

--- 
# 介绍
consul 是微服务注册中心


这里结合gin 记录一下使用案例
### 代码实例

这里存在go-micro版本问题 所以注意下载的版本是v3

```go
package main

import (
	"github.com/asim/go-micro/plugins/registry/consul/v3"
	"github.com/asim/go-micro/v3/registry"
	"github.com/asim/go-micro/v3/web"
	"github.com/gin-gonic/gin"
)

var (
	register registry.Registry
)

func init() {
	// cousul 地址默认在8500
	address := registry.Addrs("127.0.0.1:8500")
	register = consul.NewRegistry(address)
}

func InitRouter() *gin.Engine {
	router := gin.Default()
	router.GET("/v1", func(c *gin.Context) {
		c.String(200, "hello world")
	})
	return router
}

func RegisterService(engine *gin.Engine, serverName, address string) web.Service {
	microService := web.NewService(
		web.Name(serverName),
		web.Address(address),
		web.Handler(engine),
		web.Registry(register),
	)
	return microService
}

func main() {
	router:=InitRouter()
	addr := ":8080"
	serverName := "gin"
	service := RegisterService(router, serverName, addr)
	err := service.Run()
	if err != nil {
		panic(err)
	}
}
```

在官网下载 cousul 二进制文件 或者使用docker运行
```bash
docker run -itd -p 8500:8500 --name consul consul   
```


```gomod

module consul2

go 1.17

require (
	github.com/asim/go-micro/v3 v3.7.0
	github.com/gin-gonic/gin v1.7.7
)

require (
	github.com/Microsoft/go-winio v0.5.1 // indirect
	github.com/ProtonMail/go-crypto v0.0.0-20220113124808-70ae35bab23f // indirect
	github.com/acomagu/bufpipe v1.0.3 // indirect
	github.com/armon/go-metrics v0.3.10 // indirect
	github.com/asim/go-micro/plugins/registry/consul/v3 v3.7.0 // indirect
	github.com/bitly/go-simplejson v0.5.0 // indirect
	github.com/cheekybits/genny v1.0.0 // indirect
	github.com/coreos/etcd v3.3.17+incompatible // indirect
	github.com/coreos/go-systemd v0.0.0-20190719114852-fd7a80b32e1f // indirect
	github.com/coreos/pkg v0.0.0-20180928190104-399ea9e2e55f // indirect
	github.com/cpuguy83/go-md2man/v2 v2.0.1 // indirect
	github.com/emirpasic/gods v1.12.0 // indirect
	github.com/fatih/color v1.13.0 // indirect
	github.com/fsnotify/fsnotify v1.5.1 // indirect
	github.com/gin-contrib/sse v0.1.0 // indirect
	github.com/go-git/gcfg v1.5.0 // indirect
	github.com/go-git/go-billy/v5 v5.3.1 // indirect
	github.com/go-git/go-git/v5 v5.4.2 // indirect
	github.com/go-log/log v0.1.0 // indirect
	github.com/go-playground/locales v0.13.0 // indirect
	github.com/go-playground/universal-translator v0.17.0 // indirect
	github.com/go-playground/validator/v10 v10.4.1 // indirect
	github.com/gogo/protobuf v1.3.2 // indirect
	github.com/golang/protobuf v1.5.2 // indirect
	github.com/google/uuid v1.3.0 // indirect
	github.com/hashicorp/consul/api v1.12.0 // indirect
	github.com/hashicorp/go-cleanhttp v0.5.2 // indirect
	github.com/hashicorp/go-hclog v1.1.0 // indirect
	github.com/hashicorp/go-immutable-radix v1.3.1 // indirect
	github.com/hashicorp/go-rootcerts v1.0.2 // indirect
	github.com/hashicorp/golang-lru v0.5.4 // indirect
	github.com/hashicorp/serf v0.9.7 // indirect
	github.com/imdario/mergo v0.3.12 // indirect
	github.com/jbenet/go-context v0.0.0-20150711004518-d14ea06fba99 // indirect
	github.com/json-iterator/go v1.1.9 // indirect
	github.com/kevinburke/ssh_config v1.1.0 // indirect
	github.com/leodido/go-urn v1.2.0 // indirect
	github.com/lucas-clemente/quic-go v0.12.1 // indirect
	github.com/marten-seemann/qtls v0.3.2 // indirect
	github.com/mattn/go-colorable v0.1.12 // indirect
	github.com/mattn/go-isatty v0.0.14 // indirect
	github.com/micro/cli v0.2.0 // indirect
	github.com/micro/go-micro v1.16.0 // indirect
	github.com/micro/mdns v0.3.0 // indirect
	github.com/miekg/dns v1.1.46 // indirect
	github.com/mitchellh/go-homedir v1.1.0 // indirect
	github.com/mitchellh/hashstructure v1.1.0 // indirect
	github.com/mitchellh/mapstructure v1.4.3 // indirect
	github.com/modern-go/concurrent v0.0.0-20180306012644-bacd9c7ef1dd // indirect
	github.com/modern-go/reflect2 v1.0.1 // indirect
	github.com/nats-io/jwt v0.3.0 // indirect
	github.com/nats-io/nats.go v1.9.1 // indirect
	github.com/nats-io/nkeys v0.1.0 // indirect
	github.com/nats-io/nuid v1.0.1 // indirect
	github.com/nxadm/tail v1.4.8 // indirect
	github.com/oxtoacart/bpool v0.0.0-20190530202638-03653db5a59c // indirect
	github.com/patrickmn/go-cache v2.1.0+incompatible // indirect
	github.com/pkg/errors v0.9.1 // indirect
	github.com/russross/blackfriday/v2 v2.1.0 // indirect
	github.com/sergi/go-diff v1.2.0 // indirect
	github.com/shurcooL/sanitized_anchor_name v1.0.0 // indirect
	github.com/ugorji/go/codec v1.1.7 // indirect
	github.com/urfave/cli/v2 v2.3.0 // indirect
	github.com/xanzy/ssh-agent v0.3.1 // indirect
	go.uber.org/atomic v1.4.0 // indirect
	go.uber.org/multierr v1.2.0 // indirect
	go.uber.org/zap v1.10.0 // indirect
	golang.org/x/crypto v0.0.0-20220214200702-86341886e292 // indirect
	golang.org/x/mod v0.5.1 // indirect
	golang.org/x/net v0.0.0-20220127200216-cd36cc0744dd // indirect
	golang.org/x/sync v0.0.0-20210220032951-036812b2e83c // indirect
	golang.org/x/sys v0.0.0-20220209214540-3681064d5158 // indirect
	golang.org/x/text v0.3.7 // indirect
	golang.org/x/tools v0.1.9 // indirect
	golang.org/x/xerrors v0.0.0-20200804184101-5ec99f83aff1 // indirect
	google.golang.org/genproto v0.0.0-20200305110556-506484158171 // indirect
	google.golang.org/grpc v1.27.1 // indirect
	google.golang.org/protobuf v1.27.1 // indirect
	gopkg.in/tomb.v1 v1.0.0-20141024135613-dd632973f1e7 // indirect
	gopkg.in/warnings.v0 v0.1.2 // indirect
	gopkg.in/yaml.v2 v2.4.0 // indirect
)
```


该功能的Python简单实现
```python
import time

import requests
import threading
from flask import Flask

app = Flask("")


@app.route("/")
def index():
    return {
        'msg': 'lscb'
    }


def heart():
    payload = {
        'ID': '3c05bc09-21f3-4d17-b65a-334a899b59e2',
        'Name': 'micro2',
        'Tags': [
            'traefik.enable=true',
            'traefik.http.routers.micro2.rule=Host(`py.localhost`)',
        ],
        'Port': 8001, 'Address': '172.22.0.1',
        'Check': {
            'TTL': '1m0s',
            'DeregisterCriticalServiceAfter': '1m5s'
        },
        'Checks': None}

    resp = requests.put('http://127.0.0.1:8500/v1/agent/service/register', json=payload)
    while 1:
        requests.put('http://127.0.0.1:8500/v1/agent/check/pass/service:3c05bc09-21f3-4d17-b65a-334a899b59e2?note=')

        time.sleep(60)


def run():
    threading.Thread(target=heart).start()
    app.run(
        port=8001,
        host='0.0.0.0'
    )


run()

```


```python
import time
from uuid import uuid4
from signal import signal, SIGINT
from threading import Thread
from flask import Flask
from consul import Consul, Check

cli = Consul('172.17.0.2', port=8500)
port = 5000
local_host = '172.17.0.1'

service_id = str(uuid4())
app = Flask('')
cli.agent.service.register(
    name='hello-world2',
    service_id=f'{service_id}',
    address=local_host,
    port=5000,
    tags=['name:zhangsan'],
    check={
        'ttl': '10s',
        # 'http': 'http://172.17.0.1:5000/check',
        # 'interval': '5s',
        'DeregisterCriticalServiceAfter': '1m'  # 这个至少1分钟， 低于一分钟无效
    },

)


def consul_healthy_check():
    while 1:
        cli.agent.check.ttl_pass(f'service:{service_id}')
        time.sleep(5)

# 关闭服务取消注册
def close(*args):
    cli.agent.service.deregister(service_id)
    exit(0)

# 如果设置check模式为http， 就需要这个
# @app.route('/check')
# def check():
#     return {}


t = Thread(target=consul_healthy_check, daemon=True)
t.start()

signal(SIGINT, close)
app.run(host='0.0.0.0', port=port)

```
