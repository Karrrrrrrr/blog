---
date: 2024-05-01T15:00:00+08:00
title: 编译一个 Clang
categories: ["代码" , "编译",  "Clang" ]
tags: ["代码" , "Clang" ,"编译"]
---



# 编译一个Clang

## 选择一个linux发行版

### Debian

#### 换源

```shell
vim /etc/apt/sources.list
```

写入如下内容

```
# 默认注释了源码仓库，如有需要可自行取消注释
deb http://mirrors.ustc.edu.cn/debian bookworm main contrib non-free non-free-firmware
deb-src http://mirrors.ustc.edu.cn/debian bookworm main contrib non-free non-free-firmware
deb http://mirrors.ustc.edu.cn/debian bookworm-updates main contrib non-free non-free-firmware
deb-src http://mirrors.ustc.edu.cn/debian bookworm-updates main contrib non-free non-free-firmware

# backports 软件源，请按需启用
deb http://mirrors.ustc.edu.cn/debian bookworm-backports main contrib non-free non-free-firmware
deb-src http://mirrors.ustc.edu.cn/debian bookworm-backports main contrib non-free non-free-firmware
```

#### 更新源
```shell
apt update
```
#### 下载依赖
```shell
apt install -y clang++ libgmp-dev libmpfr-dev libmpc-dev make
```

#### 下载源码
```shell


```

#### 创建makefile

```shell

```

#### 编译

```shell
make -j 4
make install
```



### 解决方案

##  完整脚本

```shell

```

