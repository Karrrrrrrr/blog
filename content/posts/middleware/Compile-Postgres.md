---
date: 2024-05-05T12:00:00+08:00
title: 编译一个 Postgres
categories: ["代码" , "编译",  "Postgres" ]
tags: ["代码" , "Postgres" ,"编译"]
---


# 编译一个Postgres

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
apt -y install g++ libicu-dev pkg-config bison flex libreadline-dev zlib1g-dev make 
```

#### 下载源码
使用git
```shell
git clone  --depth=1 https://github.com/postgres/postgres.git
cd postgres
```
使用wget
```shell
wget https://mirrors.tuna.tsinghua.edu.cn/postgresql/source/v16.2/postgresql-16.2.tar.gz postgres.tar.gz
tar -xzvf postgres.tar.gz
cd postgres
```


#### 创建makefile

```shell
./configure
```

#### 编译

这个过程大约三分钟

```shell
make -j $(nproc)
```

#### 安装	

安装的位置在``` /usr/local/pgsql/bin```

```shell
make install
```

#### 初始化数据库

Postgres不允许root用户直接登录, 这里先切换到一个普通用户

这里需要保证该用户存在, 不存在需要先创建一个

```shell
su postgres
```

Postgres需要一个PGDATA的环境变量

```
export PGDATA=/home/vagrant/pgdata
```

使用initdb初始化环境

```shell
 /usr/local/pgsql/bin/initdb
```
运行Postgres
```shell
 /usr/local/pgsql/bin/postgres
```
看到如下日志, 说明编译成功了
```
2024-05-05 06:01:23.941 UTC [20109] LOG:  starting PostgreSQL 17devel on x86_64-pc-linux-gnu, compiled by gcc (Debian 12.2.0-14) 12.2.0, 64-bit
2024-05-05 06:01:23.941 UTC [20109] LOG:  listening on IPv4 address "127.0.0.1", port 5432
2024-05-05 06:01:23.941 UTC [20109] LOG:  could not bind IPv6 address "::1": Cannot assign requested address
2024-05-05 06:01:23.942 UTC [20109] LOG:  listening on Unix socket "/tmp/.s.PGSQL.5432"
2024-05-05 06:01:23.944 UTC [20112] LOG:  database system was shut down at 2024-05-05 06:01:20 UTC
2024-05-05 06:01:23.947 UTC [20109] LOG:  database system is ready to accept connections
^C2024-05-05 06:01:51.500 UTC [20109] LOG:  received fast shutdown request
2024-05-05 06:01:51.500 UTC [20109] LOG:  aborting any active transactions
2024-05-05 06:01:51.501 UTC [20109] LOG:  background worker "logical replication launcher" (PID 20115) exited with exit code 1
2024-05-05 06:01:51.501 UTC [20110] LOG:  shutting down
2024-05-05 06:01:51.502 UTC [20110] LOG:  checkpoint starting: shutdown immediate
2024-05-05 06:01:51.505 UTC [20110] LOG:  checkpoint complete: wrote 3 buffers (0.0%); 0 WAL file(s) added, 0 removed, 0 recycled; write=0.001 s, sync=0.001 s, total=0.004 s; sync files=2, longest=0.001 s, average=0.001 s; distance=0 kB, estimate=0 kB; lsn=0/14E2F60, redo lsn=0/14E2F60
2024-05-05 06:01:51.508 UTC [20109] LOG:  database system is shut down
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
apt -y install g++ libicu-dev pkg-config bison flex libreadline-dev zlib1g-dev make 
git clone  --depth=1 https://github.com/postgres/postgres.git
# wget https://mirrors.tuna.tsinghua.edu.cn/postgresql/source/v16.2/postgresql-16.2.tar.gz postgres.tar.gz && tar -xzvf postgres.tar.gz
cd postgres
./configure

# 这个编译很快 大概就三分钟
make -j $(nproc)
make install


# 切换到普通用户
# su postgresql
# export PGDATA="/home/postgresql/pgdata"
# /usr/local/pgsql/bin/initdb
# /usr/local/pgsql/bin/postgres
```



## 小技巧

在dev系下编译缺少库, 一般是``` lib{库的名字}-dev```这种格式的, 

比如icu库, 就叫 libicu-dev

readline叫: libreadline-dev

不过也有例外的, 比如zlib1g-dev