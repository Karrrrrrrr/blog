---
date: 2024-04-30T15:00:00+08:00
title: Dragonfly
categories: ["代码" , "Redis" ,"中间件"]
tags: ["代码" , "Redis" ,"中间件"]
---

# 编译一个Redis

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
```
apt install g++ make pkg-config
```

#### 下载源码
```
git clone https://github.com/redis/redis.git
```
#### 编译

```
cd redis
make -j 4 # 使用4核心编译, 加速 可以调整为本机的CPU数量
# make -j $(nproc) 这样就是最大核心数
```

耐心等待几分钟, redis编译很快的

出现如下字样 就说明编译完成了

```
root@debian12:/home/vagrant/redis# make -j 4
cd src && make all
make[1]: Entering directory '/home/vagrant/redis/src'
    CC Makefile.dep
    CC threads_mngr.o
    CC adlist.o
    CC ae.o
    CC quicklist.o
    CC anet.o
    CC dict.o
    CC kvstore.o
    CC server.o
    CC sds.o
    CC zmalloc.o
    CC lzf_c.o
    CC lzf_d.o
    CC pqsort.o
    CC zipmap.o
    CC sha1.o
    CC ziplist.o
    CC release.o
    CC networking.o
    CC util.o
    CC object.o
    CC db.o
    CC replication.o
    CC rdb.o
    CC t_string.o
    CC t_list.o
    CC t_set.o
    CC t_zset.o
    CC t_hash.o
    CC config.o
    CC aof.o
    CC pubsub.o
    CC multi.o
    CC debug.o
    CC sort.o
    CC intset.o
    CC syncio.o
    CC cluster.o
    CC cluster_legacy.o
    CC crc16.o
    CC endianconv.o
    CC slowlog.o
    CC eval.o
    CC bio.o
    CC rio.o
    CC rand.o
    CC memtest.o
    CC syscheck.o
    CC crcspeed.o
    CC crc64.o
    CC bitops.o
    CC sentinel.o
    CC notify.o
    CC setproctitle.o
    CC blocked.o
    CC hyperloglog.o
    CC latency.o
    CC sparkline.o
    CC redis-check-rdb.o
    CC redis-check-aof.o
    CC geo.o
    CC lazyfree.o
    CC module.o
    CC evict.o
    CC expire.o
    CC geohash.o
    CC geohash_helper.o
    CC childinfo.o
    CC defrag.o
    CC siphash.o
    CC rax.o
    CC t_stream.o
    CC listpack.o
    CC localtime.o
    CC lolwut.o
    CC lolwut5.o
    CC lolwut6.o
    CC acl.o
    CC tracking.o
    CC socket.o
    CC tls.o
    CC sha256.o
    CC timeout.o
    CC setcpuaffinity.o
    CC monotonic.o
    CC mt19937-64.o
    CC resp_parser.o
    CC call_reply.o
    CC script_lua.o
    CC script.o
    CC functions.o
    CC function_lua.o
    CC commands.o
    CC strl.o
    CC connection.o
    CC unix.o
    CC logreqres.o
    CC redis-cli.o
    CC redisassert.o
    CC cli_common.o
    CC cli_commands.o
    CC redis-benchmark.o
    LINK redis-server
    LINK redis-benchmark
    LINK redis-cli
    INSTALL redis-sentinel
    INSTALL redis-check-rdb
    INSTALL redis-check-aof

Hint: It's a good idea to run 'make test' ;)

make[1]: Leaving directory '/home/vagrant/redis/src'
```

src目录下 这些文件就是编译产物

```
-rwxr-xr-x 1 root root   882192 Apr 30 07:35 redis-benchmark
-rwxr-xr-x 1 root root 10645208 Apr 30 07:35 redis-check-aof
-rwxr-xr-x 1 root root 10645208 Apr 30 07:35 redis-check-rdb
-rwxr-xr-x 1 root root  1680448 Apr 30 07:35 redis-cli
-rwxr-xr-x 1 root root 10645208 Apr 30 07:35 redis-sentinel
-rwxr-xr-x 1 root root 10645208 Apr 30 07:35 redis-server
```

#### 验证

```
root@debian12:/home/vagrant/redis/src# ./redis-server
38994:C 30 Apr 2024 07:39:27.188 # WARNING Memory overcommit must be enabled! Without it, a background save or replication may fail under low memory condition. To fix this issue add 'vm.overcommit_memory = 1' to /etc/sysctl.conf and then reboot or run the command 'sysctl vm.overcommit_memory=1' for this to take effect.
38994:C 30 Apr 2024 07:39:27.188 * oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
38994:C 30 Apr 2024 07:39:27.188 * Redis version=255.255.255, bits=64, commit=f95031c4, modified=0, pid=38994, just started
38994:C 30 Apr 2024 07:39:27.188 # Warning: no config file specified, using the default config. In order to specify a config file use ./redis-server /path/to/redis.conf
38994:M 30 Apr 2024 07:39:27.189 * Increased maximum number of open files to 10032 (it was originally set to 1024).
38994:M 30 Apr 2024 07:39:27.189 * monotonic clock: POSIX clock_gettime
                _._
           _.-``__ ''-._
      _.-``    `.  `_.  ''-._           Redis 255.255.255 (f95031c4/0) 64 bit
  .-`` .-```.  ```\/    _.,_ ''-._
 (    '      ,       .-`  | `,    )     Running in standalone mode
 |`-._`-...-` __...-.``-._|'` _.-'|     Port: 6379
 |    `-._   `._    /     _.-'    |     PID: 38994
  `-._    `-._  `-./  _.-'    _.-'
 |`-._`-._    `-.__.-'    _.-'_.-'|
 |    `-._`-._        _.-'_.-'    |           https://redis.io
  `-._    `-._`-.__.-'_.-'    _.-'
 |`-._`-._    `-.__.-'    _.-'_.-'|
 |    `-._`-._        _.-'_.-'    |
  `-._    `-._`-.__.-'_.-'    _.-'
      `-._    `-.__.-'    _.-'
          `-._        _.-'
              `-.__.-'

38994:M 30 Apr 2024 07:39:27.191 * Server initialized
38994:M 30 Apr 2024 07:39:27.191 * Ready to accept connections tcp
```


可以执行以下命令,安装到全局(``` /usr/bin ```)

```shell
make install
```

## 常见报错

```
/usr/bin/ld: cannot find ../deps/jemalloc/lib/libjemalloc.a: No such file or directory
```

### 方案1

redis默认的内存分配器是jemalloc,这个在本地是没有的, 所以可以把分配器切换为libc, 加上参数重新编译即可

```shell
make MALLOC=libc -j 4
```

### 方案2

安装jemalloc, 重新编译

```shell
 apt install libjemalloc-dev
 make -j 4
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

apt update \
&& apt install -y g++ make pkg-config libjemalloc-dev \
&& git clone https://github.com/redis/redis.git \
&& cd redis \
&& make -j $(nproc)
```

