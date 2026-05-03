---
date: 2024-05-01T15:00:00+08:00
title: 编译一个 GCC
categories: ["代码" , "编译",  "GCC" ]
tags: ["代码" , "GCC" ,"编译"]
---



# 编译一个GCC

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
# git clone https://github.com/gcc-mirror/gcc.git
# cd gcc
wget http://ftp.tsukuba.wide.ad.jp/software/gcc/releases/gcc-13.2.0/gcc-13.2.0.tar.gz
tar -xzvf gcc-13.2.0.tar.gz
cd gcc-13.2.0
```

#### 创建makefile

```shell
./configure --disable-multilib
```

#### 编译

```shell
make -j 4
make install
```



这样编译的gcc编译C++文件可能会出现段错误, core dump, 找不到动态链接库之类的问题

例如以下错误

```
./a.out: /lib/x86_64-linux-gnu/libstdc++.so.6: version `GLIBCXX_3.4.32' not found (required by ./a.out)
```

原因是因为当前gcc版本高于系统的动态链接库版本, 出现不匹配

### 解决方案

1. 静态编译, 编译加上--static参数 ```g++ main.cpp --static```
2. 更新动态链接库(容易导致其他程序不兼容, 不推荐)

```shell
 # 寻找系统中合适的动态链接库
 sudo find / -name "libstdc++.so.6*"
 # 查看某个so文件
 strings /usr/lib/x86_64-linux-gnu/libstdc++.so.6 | grep GLIBCXX

 rm -rf /usr/lib/x86_64-linux-gnu/libstdc++.so.6
 ln -s /usr/local/lib64/libstdc++.so.6.0.32 /usr/lib/x86_64-linux-gnu/libstdc++.so.6
 # /usr/local/lib64/libstdc++.so.6.0.32
```

##  完整脚本



```shell
cat << EOF > /etc/apt/sources.list
# 默认注释了源码仓库，如有需要可自行取消注释
deb http://mirrors.ustc.edu.cn/debian bookworm main contrib non-free non-free-firmware
deb-src http://mirrors.ustc.edu.cn/debian bookworm main contrib non-free non-free-firmware
deb http://mirrors.ustc.edu.cn/debian bookworm-updates main contrib non-free non-free-firmware
deb-src http://mirrors.ustc.edu.cn/debian bookworm-updates main contrib non-free non-free-firmware

# backports 软件源，请按需启用
deb http://mirrors.ustc.edu.cn/debian bookworm-backports main contrib non-free non-free-firmware
deb-src http://mirrors.ustc.edu.cn/debian bookworm-backports main contrib non-free non-free-firmware
EOF

apt update 
apt install -y clang libgmp-dev libmpfr-dev libmpc-dev make
wget http://ftp.tsukuba.wide.ad.jp/software/gcc/releases/gcc-13.2.0/gcc-13.2.0.tar.gz
tar -xzvf gcc-13.2.0.tar.gz
cd gcc-13.2.0
# 编译gcc之前, 必须要有一个C/C++编译器, 可以是老版本的gcc或者是clang
./configure \
        --prefix=/usr/local \
        --enable-checking=release \
        --disable-fixed-point \
        --disable-libmpx \
        --disable-libmudflap \
        --disable-libsanitizer \
        --disable-libssp \
        --disable-libstdcxx-pch \
        --disable-multilib \
        --disable-nls \
        --disable-symvers \
        --disable-werror \
        --enable-__cxa_atexit \
        --enable-default-pie \
        --enable-languages=c,c++ \
        --enable-shared \
        --enable-threads \
        --enable-tls \
        --with-linker-hash-style=gnu \
        --with-system-zlib
# 这一步非常慢, 几个小时都是很正常的, 耐心等待即可
make -j 4
make install
```



