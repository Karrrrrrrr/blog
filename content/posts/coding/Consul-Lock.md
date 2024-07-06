---
title: "Consul Lock"
date: "2024-07-07T02:00:00+08:00"
---

# Consul分布式锁
## Golang 代码
```go
package main

import (
	"errors"
	"fmt"
	"github.com/hashicorp/consul/api"
	"sync"
	"time"
)

const (
	host = "192.168.80.128:8500"
)

var (
	Client  *api.Client
)

// 初始化consul
func initConsul() {
	client, err := api.NewClient(&api.Config{Address: host})
	if err != nil {
		panic(err)
	}
	Client = client
}

func NewLock(key, sessionTTL string) (*api.Lock, error) {
	opts := &api.LockOptions{
		Key:         key,
		SessionName: "1",
		SessionTTL:  sessionTTL,
	}

	lock, err := Client.LockOpts(opts)
	if err != nil {
		return nil, errors.New("failed to created lock")
	}
	return lock, nil
}

var wg sync.WaitGroup

func fn(key string) error {
	defer func() {
		wg.Done()
	}()
	var lock, err = NewLock(key, "10s")
	if err != nil {
		return err
	}
	_, err = lock.Lock(nil)
	if err != nil {
		return err
	}
	fmt.Println("operation for", key)
	time.Sleep(time.Second)
	err = lock.Unlock()
	if err != nil {
		return err
	}
	return nil
}

func main() {
	initConsul()

	for i := 0; i < 10; i++ {
		wg.Add(1)
		go func() {
			err := fn("id:1")
			if err != nil {
				fmt.Println(err)
			}
		}()
	}
	for i := 0; i < 10; i++ {
		wg.Add(1)
		go func() {
			err := fn("id:2")
			if err != nil {
				fmt.Println(err)
			}
		}()
	}
	wg.Wait()
}

```