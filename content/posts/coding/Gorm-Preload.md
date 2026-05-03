---
title: "Gorm Preload"
date: "2025-03-30T21:00:00+08:00"
categories: ["代码", "Golang" ]
tags: [ "Golang" ,"GORM"]
---
 


gorm 在使用Preload的时候, 在多对多模型下, 无法对中间表进行软删除判断

伪代码如下
```go

type User struct {
	ID     uint 
	UserID uint
	RoleID uint
	DeletedAt gorm.DeletedAt
	// any other field
    // code1
	Roles []Role `gorm:"->"`
	// code2
	UserRoles []UserRole `gorm:"->"`
}

type UserRole struct {
	ID     uint 
	UserID uint
	RoleID uint
	DeletedAt gorm.DeletedAt
	Role *Role
}

type Role struct {
	ID     uint 
	// any other field
}

func query() {
	var users []User
	// 这种情况下 无法对UserRole表进行软删除判断
	db.Model(User{}).Preload("Roles").Find(&users)
	
	// 解决方案1. 不使用软删除
	
	// 解决方案2. 两步Preload 不直接关联Role 实现如下
    // db.Model(User{}).Preload("UserRoles.Role").Find(&users)
}


```

### 解决方案1
中间表不使用软删除 实现如下
```go
type User struct {
	ID        uint
	UserID    uint
	RoleID    uint
	DeletedAt gorm.DeletedAt
	// any other field
	Roles []Role `gorm:"->"`
}

type UserRole struct {
	ID        uint
	UserID    uint
	RoleID    uint
}

type Role struct {
	ID uint
	// any other field
}

func query() {
	var users []User
	db.Model(User{}).Preload("Roles").Find(&users)
}

```



### 解决方案2
两步Preload 不直接关联Role 实现如下
```go
type User struct {
	ID        uint
	UserID    uint
	RoleID    uint
	DeletedAt gorm.DeletedAt
	// any other field
	// code2
	UserRoles []UserRole `gorm:"->"`
}

type UserRole struct {
	ID        uint
	UserID    uint
	RoleID    uint
	DeletedAt gorm.DeletedAt
	Role      *Role
}

type Role struct {
	ID uint
	// any other field
}

func query() {
	var users []User
	db.Model(User{}).Preload("UserRoles.Role").Find(&users)
}

```