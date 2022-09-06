---
title: "Gorm Time"
date: "2022-09-06T22:56:18+08:00"
categories: ["代码", "Golang" ]
tags: [ "Golang" ,"GORM"]
---

gorm自带的model会格式化一个不符合要求的时间， 此时就需要重写覆盖原来的方法

实现方案： 自定义一个结构体，内嵌time.Time 
且实现Value和Scan函数， 并且重写MarshalJSON函数, 覆盖原有的time.Time格式化方案

代码如下

Scan函数用于读取数据库的值， 写入结构体
Value函数用于把结构体的值转换为数据库对应结构

```go

package main

import (
	"database/sql/driver"
	"fmt"
	"time"
)

type User struct {
	Id        int
	Name      string
	Password  string
	CreatedAt JSONTime
}

type JSONTime struct {
	Time time.Time
}
// MarshalJSON on JSONTime format Time field with %Y-%m-%d %H:%M:%S

// Value insert timestamp into mysql need this function.
func (this JSONTime) Value() (driver.Value, error) {
	var zeroTime time.Time
	if this.Time.UnixNano() == zeroTime.UnixNano() {
		return nil, nil
	}
	return this.Time, nil
}

// Scan valueof time.Time
func (this *JSONTime) Scan(v interface{}) error {
	value, ok := v.(time.Time)
	if ok {
		*this = JSONTime{Time: value}
		return nil
	}
	return fmt.Errorf("can not convert %v to timestamp", v)
}

func (this *JSONTime) MarshalJSON() ([]byte, error) {
	format := this.Time.Format("\"2006-01-02 15:04:05\"")
	return []byte(format), nil
}

```
