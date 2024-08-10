
---
title: "SQL"
date: 2024-08-10T00:19:18+08:00
categories: ["代码" ]
tags: ["数据库"]
---

# SQL

## 参数化sql
手动拼接sql就会有sql注入的风险, 即使是处理了转义, 仍有可能存在漏洞, 为了解决这个问题, 通常可以使用占位符代替实际参数

值得一提的是, gorm和jdbc的参数都是在服务端处理的, psycopg2的参数是在客户端处理的, 所以仍有可能存在注入的风险
### gorm
```
var user model.User
db.Where("id = ?", 1).Find(&user)
```
### JDBC
```java
PreparedStatement preparedStatement = conn.prepareStatement("select * from user where id = ?")
preparedStatement.setInt(1,1)
```
### psycopg2
```python
cur.execute('select * from user where id = %s', 1)
```


## 预编译
预编译的原理: 向发送一段sql, 并且携带一个id, 后续就可以通过id来复用这段sql, 减少IO和sql解析的开销(其实就和声明函数有点类似,后续只需要发送函数名和参数即可调用)。

对于java而言, 下面这段代码就非常常见, 甚至是每个java初学者都写过的。
```java
PreparedStatement preparedStatement = conn.prepareStatement(sql)
```
查阅网络资料以及各种视频, 都说这段代码可以预编译,事实上是不对的, 这段代码实现预编译有一些条件
1. 必须重复调用5次以上, 才能预编译
2. 参数数量必须是固定的

对于1, 我们可以理解成前五次还是正常调用的,第六次开始才是预编译
对于2, 比如说 ```select * from users where id = ?``` 这样就是一个固定参数,就可以预编译, 如果是 ```select * from users where id in ? ```
在in里面可能是一个数组, 这样就不属于一个参数,无法预编译


### 报文
下面这段信息由抓包得来,当我们发起一个sql预编译请求,报文如下
```text
P[len]stmtcache_16be073dcd7869a2826420ad964c154767aa4547a09ffbc8 SELECT * FROM "users" WHERE id in ($1,$2,$3,$4,$5)   D   @Sstmtcache_16be073dcd7869a2826420ad964c154767aa4547a09ffbc8 S   
```
随后向服务端发送调用请求
```text
B[len]tmtcache_16be073dcd7869a2826420ad964c154767aa4547a09ffbc8  [一些参数 ... ]
```

需要说明一下, [len]表示后续的字符数量,为固定四个字节, 也就是说sql的最大长度大约是 (1<<31)