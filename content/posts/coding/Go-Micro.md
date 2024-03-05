---
title: "Go-Micro"
date: 2024-03-05T22:26:42+08:00
---

## Go-Micro-GRPC

### Github
https://github.com/go-micro/go-micro

### 命令
下载protoc
```shell
# for windows 
# https://github.com/protocolbuffers/protobuf/releases/download/v25.3/protoc-25.3-win64.zip
# for linux 
wget https://github.com/protocolbuffers/protobuf/releases/download/v25.3/protoc-25.3-linux-x86_64.zip
apt install unzip -y
unzip protoc-25.3-linux-x86_64.zip
```


设置golang源
```shell
# 可选
go env -w GOPROXY="https://proxy.golang.com.cn,direct"
# 这里还要注意设置go/bin到环境变量
```
安装go-micro命令工具
```shell
# 这里不要下载micro/micro的gen
go install github.com/go-micro/generator/cmd/protoc-gen-micro@latest
go install google.golang.org/protobuf/cmd/protoc-gen-go@latest
```

编译protobuf
```shell
protoc --proto_path=. --micro_out=. --go_out=:. proto/helloworld.proto
# protoc --proto_path=. --micro_out=. --go_out=:. proto/*.proto
```


### 坑

1. ```go-micro.dev/v4```这个依赖无法通过go mod tidy自动下载,需要手动```go get go-micro.dev/v4```
2. go-micro和micro是不同的两个包, 注意区分
3. 直接执行Makefile中的命令不一定能成功




### 代码实例
proto/helloworld.proto
```protobuf
syntax = "proto3";

package helloworld;

option go_package = "./proto;helloworld";

service HelloWorld {
	rpc Call(Request) returns (Response) {}
	rpc Stream(StreamingRequest) returns (stream StreamingResponse) {}
	rpc PingPong(stream Ping) returns (stream Pong) {}
}

message Message {
	string say = 1;
}

message Request {
	string name = 1;
}

message Response {
	string msg = 1;
}

message StreamingRequest {
	int64 count = 1;
}

message StreamingResponse {
	int64 count = 1;
}

message Ping {
	int64 stroke = 1;
}

message Pong {
	int64 stroke = 1;
}
```


main.go 
```go
package main

import (
	"go-micro.dev/v4"
	"go-micro.dev/v4/logger"
	"helloworld/handler"
	pb "helloworld/proto"
)

func main() {

	// Create service
	srv := micro.NewService(
		micro.Name("helloworld"),
		// micro.Registry(registry.EtcdRegistry),
		micro.Address(":8080"),
	)

	// Register handler
	pb.RegisterHelloWorldHandler(srv.Server(), handler.New())

	// Run service
	if err := srv.Run(); err != nil {
		logger.Fatal(err)
	}
}
```




```go
package handler

import (
	"context"
	"fmt"
	//log "micro.dev/v4/service/logger"

	pb "helloworld/proto"
)

type HelloWorld struct{}

// Return a new handler
func New() *HelloWorld {
	return &HelloWorld{}
}

// Call is a single request handler called via client.Call or the generated client code
func (e *HelloWorld) Call(ctx context.Context, req *pb.Request, rsp *pb.Response) error {
	fmt.Println("Received HelloWorld.Call request")
	rsp.Msg = "Hello " + req.Name
	return nil
}

// Stream is a server side stream handler called via client.Stream or the generated client code
func (e *HelloWorld) Stream(ctx context.Context, req *pb.StreamingRequest, stream pb.HelloWorld_StreamStream) error {
	fmt.Printf("Received HelloWorld.Stream request with count: %d", req.Count)

	for i := 0; i < int(req.Count); i++ {
		fmt.Printf("Responding: %d", i)
		if err := stream.Send(&pb.StreamingResponse{
			Count: int64(i),
		}); err != nil {
			return err
		}
	}

	return nil
}

// PingPong is a bidirectional stream handler called via client.Stream or the generated client code
func (e *HelloWorld) PingPong(ctx context.Context, stream pb.HelloWorld_PingPongStream) error {
	for {
		req, err := stream.Recv()
		if err != nil {
			return err
		}
		fmt.Printf("Got ping %v", req.Stroke)
		if err := stream.Send(&pb.Pong{Stroke: req.Stroke}); err != nil {
			return err
		}
	}
}

```

### 使用etcd作为服务发现中心
```go
package registry

import (
	"github.com/go-micro/plugins/v4/registry/etcd"
	reg "go-micro.dev/v4/registry"
)

var (
	Address      = reg.Addrs("127.0.0.1:2380")
	EtcdRegistry = etcd.NewRegistry(
		Address,
	)
)
```

### 客户端发现服务
```go
func GetServiceAddr(serviceName string) (address string, err error) {
	//var retryCount int
	for retryCount := 0; retryCount < 5; retryCount++ {
		servers, err := etcdRegistry.GetService(serviceName)
		if err != nil {
			continue
		}
		//var services []*registry.Service
		//for _, value := range servers {
		//	services = append(services, value)
		//}
		next := selector.RoundRobin(servers)
		for node, err := next(); err == nil; {
			address = node.Address
		}

		if len(address) > 0 {
			return address, nil
		}
		time.Sleep(time.Second * 1)
	}
	return "", errors.New("not found")
}
```


```go
package main

import (
	"context"
	"fmt"
	"github.com/go-micro/starter/registry"
	"go-micro.dev/v4"
	pb "helloworld/proto"
)

func main() {

	service := micro.NewService(
		micro.Name("helloworld"),
		micro.Registry(registry.EtcdRegistry),
	)

	//service.Init()

	cli := pb.NewExampleService("starter", service.Client())

	rsp, err := cli.Call(context.TODO(), &pb.CallRequest{Name: "Kar"})
	if err != nil {
		fmt.Println(err)
	}

	fmt.Println(rsp.Message)
}

```


## Go-Micro-Gin

这种方式可以绕开protoc, 好处就是多了服务发现功能,调用方式和http一样
```go
package main

import (
	"flag"
	"fmt"
	"github.com/gin-gonic/gin"
	"github.com/go-micro/plugins/v4/registry/etcd"
	micro "go-micro.dev/v4"
	registry "go-micro.dev/v4/registry"
	"go-micro.dev/v4/selector"
	web "go-micro.dev/v4/web"
	"time"
)

var (
	port         = flag.String("port", "8080", "")
	address      = registry.Addrs("127.0.0.1:2379")
	etcdRegistry = etcd.NewRegistry(address)
)
var (
	_ = micro.Address
)

func GetServiceAddr(serviceName string) (address string) {
	var retryCount int
	for {
		servers, err := etcdRegistry.GetService(serviceName)
		if err != nil {
			fmt.Println(err.Error())
		}
		var services []*registry.Service
		for _, value := range servers {
			fmt.Println(value.Name, ":", value.Version)
			services = append(services, value)
		}
		next := selector.RoundRobin(services)
		if node, err := next(); err == nil {
			address = node.Address
		}
		if len(address) > 0 {
			return
		}
		//重试次数++
		retryCount++
		time.Sleep(time.Second * 1)
		//重试5次为获取返回空
		if retryCount >= 5 {
			return
		}
	}
}
func NewRouter() *gin.Engine {
	var app = gin.Default()
	app.GET("/", func(c *gin.Context) {
		c.JSON(200, gin.H{"message": "hello world"})
	})
	return app

}

func main() {
	flag.Parse()
	router := NewRouter()
	service := web.NewService(
		web.Name("starter"),
		web.Handler(router),
		web.Registry(etcdRegistry),
		web.Address(":"+*port), // 这个可以不指定, 使用随机端口
	)

	if err := service.Run(); err != nil {
		panic(err)
	}

}
```

## client
自己实现一个客户端,负责动态转发请求
```go
package main

import (
	"errors"
	"flag"
	"fmt"
	etcd "github.com/go-micro/plugins/v4/registry/etcd"
	micro "go-micro.dev/v4"
	registry "go-micro.dev/v4/registry"
	"go-micro.dev/v4/selector"
	"io"
	"net"
	"time"
)

var (
	port         = flag.String("port", "8080", "")
	address      = registry.Addrs("127.0.0.1:2379")
	etcdRegistry = etcd.NewRegistry(address)
)
var (
	_ = micro.Address
)

func GetServiceAddr(serviceName string) (address string, err error) {
	//var retryCount int
	for retryCount := 0; retryCount < 5; retryCount++ {
		servers, err := etcdRegistry.GetService(serviceName)
		if err != nil {
			continue
		}
		//var services []*registry.Service
		//for _, value := range servers {
		//	services = append(services, value)
		//}
		next := selector.RoundRobin(servers)
		for node, err := next(); err == nil; {
			address = node.Address
		}

		if len(address) > 0 {
			return address, nil
		}
		time.Sleep(time.Second * 1)
	}
	return "", errors.New("not found")
}

func Handle(conn net.Conn) {
	addr, err := GetServiceAddr("starter")
	fmt.Println("to: ", addr)
	remote, err := net.Dial("tcp", addr)
	if err != nil {
		return
	}
	go io.Copy(remote, conn)
	go io.Copy(conn, remote)
}

func main() {
	var server, err = net.Listen("tcp", ":5000")
	if err != nil {
		panic(err)
	}
	// 这里可以换成 reverse_proxy,也可以自定义拦截器
	for {
		cli, err := server.Accept()
		if err != nil {
			continue
		}
		go Handle(cli)
	}
}
```
