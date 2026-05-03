---
date: 2025-09-04T16:30:00+08:00

title: Consul KV数据修复
categories: ["工具"]
tags: ["Consul"]
---

意外断电等情况下,raft.db文件会丢失数据,如果数据完整 但是格式异常,此时可以尝试手动修复(不保证一定可以恢复)
```go
package main

import (
	"io"
	"os"
	"regexp"
    "fmt"
)

func main() {
	file, err := os.Open("./raft.db")
	if err != nil {
		panic(err)
	}

	// Value后面有3个字符,所以需要...吃掉这三个字符
	// kv结束后会有?Op结尾
	compile, err := regexp.Compile(`(?s)Key.(.*?/.*?).LockIndex.*?Value...(.*?).Op`)

	all, err := io.ReadAll(file)
	submatch := compile.FindAllSubmatch(all, -1)
	var mp = make(map[string][]string)
	var newMap = make(map[string]string)

	for _, item := range submatch {
		mp[string(item[1])] = append(mp[string(item[1])], string(item[2]))
		newMap[string(item[1])] = string(item[2])
	}
	for s, v := range newMap {
		fmt.Println(s, v)
	}
}

```