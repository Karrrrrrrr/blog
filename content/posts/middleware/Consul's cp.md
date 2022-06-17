---
date: 2022-03-25T00:53:09+08:00
title: Consul注册微服务的一些方案 
cover: https://images2.alphacoders.com/227/thumbbig-227910.webp
categories: ["代码" , "微服务" ,"中间件"]
tags: ["代码" , "微服务" ,"中间件"]

--- 
# 介绍
Consul是一个服务发现的工具，微服务可以注册到Consul从而实现动态插拔服务，而不用进行修改配置或者重启服务器，等一系列操作。

但是Consul只是一个服务发现的组件，它本身无法进行转发流量，不过好在Consul提供了DNS服务

[comment]: <> (那么有以下解决方案)
## 解决方案

1. 修改服务器DNS解析指向Consul,这里有坑，nginx的解析只能取到ip不能取到端口，所以所有的服务都得用80端口(Docker部署的话不会有特别大影响，但是稍微麻烦了点)

2. fabio代理Consul，这是一个官方提供的工具，专门用于转发Consul的流量，也只能用来转发Consul(官方推荐，但是未做测试)

3. 动态读取Consul的服务，编写一个模板，实时修改nginx的配置文件并重启

4.  用Kong网关，配置(/etc/kong/kong.conf)DNS指向Consul，这里DNS必须是Ip而不能用域名，配置DNS之后无法用域名连接，所以提前查询PG所在的域名
Docker的DNS一般是127.0.0.11，所以配上就不存在上面的问题
```
# /etc/kong/kong.conf
dns_resolver=115.29.227.177:8600,127.0.0.11,8.8.8.8
```

[comment]: <> (方案四是最推荐的，只是配置Kong比较麻烦)

5. Traefik + Docker 天然的服务发现 

6. Consul + Traefik  
在traefik里面配置consul catelog， traefik能自动读取consul的注册节点从而反代。


推荐方案六， 相对而言最简单

