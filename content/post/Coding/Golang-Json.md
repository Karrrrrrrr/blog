---
title: GolangJSON序列化技巧
cover: https://tse3-mm.cn.bing.net/th/id/OIP-C.6WNXKZEe6OrhwI2x-JzRqQHaDt?pid=ImgDet&rs=1
date: 2022-05-08T18:56:46+08:00
---

### GolangJSON序列化技巧
当我们从数据库里取出数据之后，通常会包装成JSON字符串传给前端。

这里通常有个问题， 假如ID是雪花算法生成的， ID会超过int的范围， 从而在前端丢失进度
因为JS数字最大是1<<53

为了避免这种问题， 最好在传给前端的时候 转化为string



```go
// 将[]int 序列化为[]string
// 这里就需要用到自定义序列化，也就是实现json/encode 里面的MarshalJson和UnmarshalJson方法。

package main

import (
	"encoding/json"
	"fmt"
	"strconv"
	"strings"
)
 

type IntString int64

type RequestParam struct {
	LoadWaybills []IntString `json:"load_waybills"`
}

func (i IntString) MarshalJSON() ([]byte, error) {
	return []byte(fmt.Sprintf("\"%v\"", i)), nil
}

func (i *IntString) UnmarshalJSON(value []byte) error {
	m, err := strconv.ParseInt(string(value[1:len(value)-1]), 10, 32)
	if err != nil {
		return err
	}
	*i = IntString(m)
	return nil
}

// 以下同理
//type StringInt string
//
//func (i StringInt) MarshalJSON() ([]byte, error) {
//  val, err := strconv.Atoi(string(i))
//  
//  return []byte(fmt.Sprintf("\"%v\"", i)), nil
//}
//
//func (i *StringInt) UnmarshalJSON(value []byte) error {
//  m, err := strconv.ParseInt(string(value[1:len(value)-1]), 10, 32)
//  if err != nil {
//    return err
//  }
//  *i = IntString(m)
//  return nil
//}

```