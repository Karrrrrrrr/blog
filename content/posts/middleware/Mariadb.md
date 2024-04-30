---
date: 2024-04-30T17:00:00+08:00
title: 编译一个 Mariadb
categories: ["代码" , "Mariadb" ,"中间件"]
tags: ["代码" , "Mariadb" ,"中间件"]
---

# 编译一个Maraidb

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
git clone https://github.com/MariaDB/server.git maraidb-server
```
#### 创建编译配置文件

这个过程大约3~5分钟

```shell
cd  maraidb-server \
&& cmake . -DCMAKE_INSTALL_PREFIX=/usr/local/mysql -DMYSQL_DATADIR=/data/mysql -DSYSCONFDIR=/etc -DWITHOUT_TOKUDB=1 -DWITH_INNOBASE_STORAGE_ENGINE=1 -DWITH_ARCHIVE_STPRAGE_ENGINE=1 -DWITH_BLACKHOLE_STORAGE_ENGINE=1 -DWIYH_READLINE=1 -DWIYH_SSL=system -DVITH_ZLIB=system -DWITH_LOBWRAP=0 -DMYSQL_UNIX_ADDR=/tmp/mysql.sock -DDEFAULT_CHARSET=utf8 -DDEFAULT_COLLATION=utf8_general_ci
```

耐心等待几分钟, 出现如下字样 就说明编译配置(Makefile)完成了

```
-- The following OPTIONAL packages have not been found:

 * Boost (required version >= 1.81.0)
 * LibXml2
 * Java (required version >= 1.6)
   Required for the CONNECT_JDBC feature
 * JNI
   Required for the CONNECT_JDBC feature
 * PMEM
 * GSSAPI
 * BZip2
 * LZ4 (required version >= 1.6)
 * LibLZMA
 * LZO
 * Snappy

-- Configuring done
-- Generating done
-- Build files have been written to: /home/vagrant/mariadb-server
```

#### 编译

这个过程大约10分钟, 视配置而定

```shell
make -j  $(nproc)
```

#### 验证

```shell
vagrant@debian12:~/mariadb-server$ find ./ | grep sql/mariadbd
# 返回如下就表示编译成功
./sql/mariadbd
```


可以执行以下命令,安装到全局, 这个路径由 ```DCMAKE_INSTALL_PREFIX```指定

```shell
make install
```

## 常见报错

### 缺少 libcurses

执行

```shell
apt install libcurses-ocaml-dev
```
报错信息
```
CMake Error at /usr/share/cmake-3.25/Modules/FindPackageHandleStandardArgs.cmake:230 (message):
  Could NOT find Curses (missing: CURSES_LIBRARY CURSES_INCLUDE_PATH)
Call Stack (most recent call first):
  /usr/share/cmake-3.25/Modules/FindPackageHandleStandardArgs.cmake:600 (_FPHSA_FAILURE_MESSAGE)
  /usr/share/cmake-3.25/Modules/FindCurses.cmake:268 (FIND_PACKAGE_HANDLE_STANDARD_ARGS)
  cmake/readline.cmake:55 (FIND_PACKAGE)
  cmake/readline.cmake:188 (FIND_CURSES)
  CMakeLists.txt:400 (MYSQL_CHECK_READLINE)
```

### 缺少GnuTLS

执行

```shell
apt install gnutls-dev
```

报错信息

```
CMake Error at /usr/share/cmake-3.25/Modules/FindPackageHandleStandardArgs.cmake:230 (message):
  Could NOT find GnuTLS (missing: GNUTLS_LIBRARY GNUTLS_INCLUDE_DIR)
  (Required is at least version "3.3.24")
Call Stack (most recent call first):
  /usr/share/cmake-3.25/Modules/FindPackageHandleStandardArgs.cmake:600 (_FPHSA_FAILURE_MESSAGE)
  /usr/share/cmake-3.25/Modules/FindGnuTLS.cmake:68 (FIND_PACKAGE_HANDLE_STANDARD_ARGS)
  libmariadb/CMakeLists.txt:335 (FIND_PACKAGE)

```



### 缺少 zstd

```shell
apt install libzstd-dev
```

### 缺少 curl

```shell
apt install libcurl4-gnutls-dev
```

### 缺少 libpcre

```shell
apt install libpcre2-dev
```



### 缺少 bison

执行(这个不需要dev包,直接安装即可)

```shell
apt install bison
```

报错信息

```
CMake Error at sql/CMakeLists.txt:390 (MESSAGE):
  Bison (GNU parser generator) is required to build MySQL.Please install
  bison.
```

### 找不到-lz

```shell
apt install zlib1g-dev
```

报错如下

```
/usr/bin/ld: cannot find -lz: No such file or directory
```



##  完整脚本(使用git)

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
# 看情况配置git代理
# git config --global http.proxy http://192.168.150.1:8889
# 或者去gitee下载, 但不保证是最新版本
# git clone https://gitee.com/mirrors/mariadb.git

apt update \
&& apt install -y g++ make cmake zlib1g-dev bison libpcre2-dev gnutls-dev libcurl4-gnutls-dev libcurses-ocaml-dev\
&& git clone https://github.com/MariaDB/server.git maraidb-server \
&& cd maraidb-server \
&& cmake .\
    -DCMAKE_INSTALL_PREFIX=/usr/local/mysql \
    -DMYSQL_DATADIR=/data/mysql \
    -DSYSCONFDIR=/etc \
    -DWITHOUT_TOKUDB=1 \
    -DWITH_INNOBASE_STORAGE_ENGINE=1 \
    -DWITH_ARCHIVE_STPRAGE_ENGINE=1 \
    -DWITH_BLACKHOLE_STORAGE_ENGINE=1 \
    -DWIYH_READLINE=1 \
    -DWIYH_SSL=system \
    -DVITH_ZLIB=system \
    -DWITH_LOBWRAP=0 \
    -DMYSQL_UNIX_ADDR=/tmp/mysql.sock \
    -DDEFAULT_CHARSET=utf8 \
    -DDEFAULT_COLLATION=utf8_general_ci \
&& make -j $(nproc)
```



## 完整脚本(压缩包)

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
&& apt install -y g++ make wget cmake tar zlib1g-dev bison libpcre2-dev gnutls-dev libcurl4-gnutls-dev libcurses-ocaml-dev\
&& wget -nc https://mirrors.ustc.edu.cn/mariadb/mariadb-11.5.0/source/mariadb-11.5.0.tar.gz \
&& tar -xzvf mariadb-11.5.0.tar.gz \
&& cd mariadb-11.5.0 \
&& cmake .\
    -DCMAKE_INSTALL_PREFIX=/usr/local/mysql \
    -DMYSQL_DATADIR=/data/mysql \
    -DSYSCONFDIR=/etc \
    -DWITHOUT_TOKUDB=1 \
    -DWITH_INNOBASE_STORAGE_ENGINE=1 \
    -DWITH_ARCHIVE_STPRAGE_ENGINE=1 \
    -DWITH_BLACKHOLE_STORAGE_ENGINE=1 \
    -DWIYH_READLINE=1 \
    -DWIYH_SSL=system \
    -DVITH_ZLIB=system \
    -DWITH_LOBWRAP=0 \
    -DMYSQL_UNIX_ADDR=/tmp/mysql.sock \
    -DDEFAULT_CHARSET=utf8 \
    -DDEFAULT_COLLATION=utf8_general_ci \
&& make -j $(nproc)
```

