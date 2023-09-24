---
title: "开发换源操作"
date: "2023-09-24T21:00:00+08:00"
categories: ["代码","CDN" ]
tags: ["CDN"]
---
 

## Vite加速
<a href="/posts/coding/vite-cdn">点击此处</a>


## maven 
修改 ```settings.xml``` 
```xml
<?xml version="1.0" encoding="UTF-8"?>
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">
    <mirrors>
        <mirror>
            <id>aliyunmaven</id>
            <mirrorOf>*</mirrorOf>
            <name>alicloud</name>
            <url>https://maven.aliyun.com/repository/public</url>
        </mirror>
    </mirrors>
</settings>
```

## gradle 



修改 ```build.gradle```
```gradle 
buildscript {
    repositories {
        maven {url 'https://maven.aliyun.com/repository/google'}
        maven {url 'https://maven.aliyun.com/repository/gradle-plugin'}
        maven {url 'https://maven.aliyun.com/repository/public'}
    }
    dependencies {
        // ... 
    }
}

allprojects {
    repositories {
        maven {url 'https://maven.aliyun.com/repository/google'}
        maven {url 'https://maven.aliyun.com/repository/gradle-plugin'}
        maven {url 'https://maven.aliyun.com/repository/public'}
    }
}
```

## pip 
### 阿里云
```shell    
pip config set global.index-url https://mirrors.aliyun.com/pypi/simple/
pip config set install.trusted-host mirrors.aliyun.com
```
### 清华
```shell
pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
pip config set install.trusted-host pypi.tuna.tsinghua.edu.cn
```

## yarn
### 设置
```shell
yarn config set registry 'https://registry.npmmirror.com'
```
### 撤销
```shell
yarn config set registry https://registry.yarnpkg.com
```
### 删除
```shell
yarn config delete registry
```


## pnpm / npm 
pnpm和npm会共用一个地址, 所以设置一个即可
```shell
pnpm config set registry https://registry.npmmirror.com
```



## golang 
golang推荐直接修改环境变量 goproxy
### powershell 
```powershell
setx GOPROXY "https://goproxy.cn,direct"
```
### linux 
在linux中推荐 ```.bashrc```中写入如下命令
```shell
export GOPROXY="https://goproxy.cn,direct"
```


### bash/cmd/pwsh
这种方式会被环境变量覆盖, 优先级不高,不如以上命令
```shell    
go env -w https://goproxy.cn,direct
```

## git

```shell
git clone --config "http.proxy='http://127.0.0.1:{port}'" --depth=1 ... 
```

## cli代理设置
一般来说, 程序都会读取环境变量中 "http_proxy"和"https_proxy"作为代理服务器
### powershell
```powershell
$env:http_proxy=""
```
### cmd
```shell
set https_proxy=""
# 读取 echo %https_proxy%
```
### bash 
```shell
export https_proxy=""
```

### pnpm 
```shell
pnpm config set https-proxy ""
```



## 一些官方镜像站
### 阿里云
https://developer.aliyun.com/mirror/?spm=a2c6h.13651102.0.0.568d1b11MAxjf9&serviceType=mirror
### 清华
https://mirrors.tuna.tsinghua.edu.cn/
### 腾讯
https://mirrors.tencent.com/