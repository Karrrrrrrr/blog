---
title: "Traefik"
date: 2022-05-10T20:00:39+08:00 
cover: https://doc.traefik.io/traefik/assets/img/traefik-architecture.png
categories: ["微服务" ,"中间件"]
tags: ["微服务" ,"中间件","Proxy","网关"]

---
# 介绍
反向代理工具， 但是无法直接作为静态文件服务器

## 静态配置参考
```yaml
api:
  dashboard: true

entryPoints:
  http:
    address: ":80"
  https:
    address: ":443"

providers:
  docker:
    endpoint: "unix:///var/run/docker.sock"
    exposedByDefault: false
  file:
    filename: /configurations/dynamic.yml
  consul:
    endpoints:
      - consul:8500
    rootKey: traefik
  consulCatalog:
    endpoint:
      address: consul:8500
```


## 动态配置参考
```yaml
http:
  routers:
  middlewares:
    path-rewrite:
      replacepathregex:
        regex: "/v1/(.*)/"
        replacement: "/"
    gzip:
      compress: true
tls:
  options:
    default:
      cipherSuites:
        - TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384
        - TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
        - TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256
        - TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256
        - TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305
        - TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305
      minVersion: VersionTLS12
```



## 中间件
| 中间件    | 目的 |    区域 |
|  ----   | ----  | --- | 
| 添加前缀 (AddPrefix)  |    添加路径前缀 | 路径修改器 | 
| 基本认证 (BasicAuth)  |    添加基本 | 身份验证 安全、认证  | 
| 缓冲 (Buffering)  |    缓冲请求 | /响应 请求生命周期 |
| 链 (Chain)  |    组合多个中间件 |    杂项 |
| 断路器 (CircuitBreaker)  |    防止调用不健康的服务 |    请求生命周期 |
| 压缩 (Compress)  |    压缩响应 |    内容修饰符 |
| 内容类型 (ContentType)  |    处理 |  Content-Type 自动检测 杂项 |
| 摘要认证 (DigestAuth)  |    添加摘要身份验证 |    安全、认证 |
| 错误 (Errors)     |    定义自定义错误页面 |    请求生命周期 |
| 转发验证 (ForwardAuth)           |    代表身份验证 |    安全、认证 |
| 标头 (Headers)  |    添加          | /更新标题 安全 |
| IP白名单 (IPWhiteList)              |    限制允许的客户端 |  IP 安全、请求生命周期 |
| 飞行请求 (InFlightReq)           |    限制同时连接的数量 |    安全、请求生命周期 |
| 通过TLSClientCert (PassTLSClientCert)  |    在标头中添加客户端证书 |    安全 |
| 速率限制 (RateLimit)            |    限制呼叫频率 |    安全、请求生命周期 |
| 重定向方案 (RedirectScheme)           |    基于方案的重定向 |    请求生命周期 |
| 重定向正则表达式 (RedirectRegex)       |    基于正则表达式的重定向 |    请求生命周期 |
| 替换路径 (ReplacePath) |    更改请求的路径 |    路径修改器 |
| 替换路径正则表达式 (ReplacePathRegex)   |    更改请求的路径 |    路径修改器 |
| 重试 (Retry)  |    发生错误时自动重试 |    请求生命周期 |
| 带前缀 (StripPrefix)  |    更改请求的路径 |    路径修改器 |
| 带前缀正则表达式 (StripPrefixRegex)  |    更改请求的路径 |    路径修改器 |