---
title: "开发换源操作"
date: "2023-09-24T21:00:00+08:00"
categories: ["代码","CDN" ]
tags: ["CDN"]
---
 

## Vite加速
<div>
    <a href="/posts/coding/vite-cdn" style="border-bottom: 1px solid skyblue; color: deepskyblue">传送门</a>
</div>


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
### gradle of SpringBoot
```gradle 
plugins {
    id 'java'
    id 'org.springframework.boot' version '3.1.4'
    id 'io.spring.dependency-management' version '1.1.3'
}

group = 'com.example'
version = '0.0.1-SNAPSHOT'

java {
    sourceCompatibility = '17'
}

configurations {
    compileOnly {
        extendsFrom annotationProcessor
    }
}

repositories {
    maven { url 'https://maven.aliyun.com/repository/google' }
    maven { url 'https://maven.aliyun.com/repository/gradle-plugin' }
    maven { url 'https://maven.aliyun.com/repository/public' }
    mavenCentral()
}

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-jdbc'
    implementation 'org.springframework.boot:spring-boot-starter-web'
    compileOnly 'org.projectlombok:lombok'
    developmentOnly 'org.springframework.boot:spring-boot-devtools'
    runtimeOnly 'org.postgresql:postgresql'
    annotationProcessor 'org.projectlombok:lombok'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
}

tasks.named('test') {
    useJUnitPlatform()
}
```



## pip 

### aliyun
```shell    
pip config set global.index-url https://mirrors.aliyun.com/pypi/simple/
pip config set install.trusted-host mirrors.aliyun.com
```

### tsinghua
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

## set proxy of cli
一般来说, 程序都会读取环境变量中 "http_proxy"和"https_proxy"作为代理服务器
### powershell
```powershell
$env:http_proxy=""
```
### cmd
```shell
set https_proxy=""
# read echo %https_proxy%
```
### bash 
```shell
export https_proxy=""
```

### pnpm 
```shell
pnpm config set https-proxy ""
```


## Alpine 

```shell
sed -i 's/dl-cdn.alpinelinux.org/mirrors.aliyun.com/g' /etc/apk/repositories
sed -i 's/dl-cdn.alpinelinux.org/mirrors.ustc.edu.cn/g' /etc/apk/repositories
```



## Any Mirrors
### Aliyun
https://developer.aliyun.com/mirror/
### Tsinghua
https://mirrors.tuna.tsinghua.edu.cn/
### Tencent
https://mirrors.tencent.com/