---
date: 2024-04-29T15:00:00+08:00
title: 编译一个 Dragonfly
categories: ["代码" , "Redis" ,"中间件"]
tags: ["Redis" ,"编译","数据库"]
---

# 编译一个Dragonfly

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



```shell
apt install update
git clone "https://github.com/dragonflydb/dragonfly.git" && cd dragonfly && git submodule update --init --recursive

# make中包含gcc,但是没有g++
apt install cmake make g++ ninja-build zlib1g-dev libboost-dev  libzstd-dev  libboost-context-dev  openssl libssl-dev bison dh-autoreconf
```

## 编译时可能遇到的问题

#### 缺少zlib

执行

```shell
apt install zlib1g-dev
```

报错日志
```
CMake Error at /usr/share/cmake-3.25/Modules/CMakeTestCCompiler.cmake:70 (message):
  The C compiler

    "/usr/bin/gcc"

  is not able to compile a simple test program.

  It fails with the following output:

    Change Dir: /home/vagrant/dragonfly/build-release/CMakeFiles/CMakeScratch/TryCompile-z5y99i

    Run Build Command(s):/usr/bin/ninja cmTC_0fe8f && [1/2] Building C object CMakeFiles/cmTC_0fe8f.dir/testCCompiler.c.o
    [2/2] Linking C executable cmTC_0fe8f
    FAILED: cmTC_0fe8f
    : && /usr/bin/gcc  -lz -static-libstdc++ -static-libgcc CMakeFiles/cmTC_0fe8f.dir/testCCompiler.c.o -o cmTC_0fe8f   && :
    /usr/bin/ld: cannot find -lz: No such file or directory
    collect2: error: ld returned 1 exit status
    ninja: build stopped: subcommand failed.

  CMake will not be able to correctly generate this project.
Call Stack (most recent call first):
  CMakeLists.txt:14 (project)

```



#### 缺少C++编译器

执行

```shell
apt install g++
```

报错日志
```
CMake Error at CMakeLists.txt:14 (project):
  The CMAKE_CXX_COMPILER:

    /usr/bin/g++

  is not a full path to an existing compiler tool.

  Tell CMake where to find the compiler by setting either the environment
  variable "CXX" or the CMake cache entry CMAKE_CXX_COMPILER to the full path
  to the compiler, or to the compiler name if it is in the PATH
```



#### 缺少boost库
执行
```shell
apt install libboost-dev
```

报错日志
```
CMake Error at /usr/share/cmake-3.25/Modules/FindPackageHandleStandardArgs.cmake:230 (message):
  Could NOT find Boost (missing: Boost_INCLUDE_DIR context system) (Required
  is at least version "1.71.0")
Call Stack (most recent call first):
  /usr/share/cmake-3.25/Modules/FindPackageHandleStandardArgs.cmake:600 (_FPHSA_FAILURE_MESSAGE)
  /usr/share/cmake-3.25/Modules/FindBoost.cmake:2376 (find_package_handle_standard_args)
  helio/cmake/third_party.cmake:245 (find_package)
  CMakeLists.txt:52 (include)
```

#### 缺少boostcontext库
执行
```shell
apt install libboost-context-dev
```

报错日志
```
CMake Error at /usr/lib/x86_64-linux-gnu/cmake/Boost-1.74.0/BoostConfig.cmake:141 (find_package):
  Could not find a package configuration file provided by "boost_context"
  (requested version 1.74.0) with any of the following names:

    boost_contextConfig.cmake
    boost_context-config.cmake

  Add the installation prefix of "boost_context" to CMAKE_PREFIX_PATH or set
  "boost_context_DIR" to a directory containing one of the above files.  If
  "boost_context" provides a separate development package or SDK, be sure it
  has been installed.
Call Stack (most recent call first):
  /usr/lib/x86_64-linux-gnu/cmake/Boost-1.74.0/BoostConfig.cmake:258 (boost_find_component)
  /usr/share/cmake-3.25/Modules/FindBoost.cmake:594 (find_package)
  helio/cmake/third_party.cmake:245 (find_package)
  CMakeLists.txt:52 (include)

```

#### 缺少zstd库

```shell
apt install libzstd-dev
```

```
CMake Error at src/server/CMakeLists.txt:72 (find_library):
  Could not find ZSTD_LIB using the following names: libzstd.a,
  libzstdstatic.a, zstd

```
执行
```shell
apt install openssl libssl-dev
```


报错日志

```
CMake Error at helio/cmake/internal.cmake:144 (target_link_libraries):
  Target "tls_lib" links to:

    OpenSSL::SSL

  but the target was not found.  Possible reasons include:

    * There is a typo in the target name.
    * A find_package call is missing for an IMPORTED target.
    * An ALIAS target is missing.

Call Stack (most recent call first):
  helio/util/tls/CMakeLists.txt:5 (cxx_link)


CMake Error at helio/cmake/internal.cmake:144 (target_link_libraries):
  Target "awsv2_lib" links to:

    OpenSSL::Crypto

  but the target was not found.  Possible reasons include:

    * There is a typo in the target name.
    * A find_package call is missing for an IMPORTED target.
    * An ALIAS target is missing.

Call Stack (most recent call first):
  helio/util/aws/CMakeLists.txt:14 (cxx_link)


CMake Error at helio/cmake/internal.cmake:144 (target_link_libraries):
  Target "dfly_core" links to:

    OpenSSL::Crypto

  but the target was not found.  Possible reasons include:

    * There is a typo in the target name.
    * A find_package call is missing for an IMPORTED target.
    * An ALIAS target is missing.

Call Stack (most recent call first):
  src/core/CMakeLists.txt:12 (cxx_link)
```

#### 缺少autoreconf库

```shell
apt install dh-autoreconf
```

报错日志
```
CMake Error at /home/vagrant/dragonfly/build-release/third_party/src/gperf_project-stamp/gperf_project-patch-Release.cmake:49 (message):
  Command failed: No such file or directory

   'autoreconf' '-i'

  See also

    /home/vagrant/dragonfly/build-release/third_party/src/gperf_project-stamp/gperf_project-patch-*.log
```
#### 缺少bison

执行

```shell
apt install bison
```

报错日志

```
FAILED: /home/vagrant/dragonfly/genfiles/src/core/search/parser.cc /home/vagrant/dragonfly/genfiles/src/core/search/parser.hh
cd /home/vagrant/dragonfly/src/core/search && mkdir -p /home/vagrant/dragonfly/genfiles/src/core/search && bison --language=c++ -o /home/vagrant/dragonfly/genfiles/src/core/search/parser.cc parser.y -Wconflicts-sr
/bin/sh: 1: bison: not found
```

#### aws-project-gitclone报错

这是因为他会在编译期间下载aws包, 多半会下载失败, 建议配上git代理,然后重新编译 多试几次即可

```
 git config --global http.proxy http://192.168.150.1:8889
```



```
CMake Error at src/aws_project-stamp/aws_project-download-Release.cmake:49 (message):
  Command failed: 1

   '/usr/bin/cmake' '-P' '/home/vagrant/dragonfly/build-release/third_party/tmp/aws_project-gitclone.cmake'

  See also

    /home/vagrant/dragonfly/build-release/third_party/src/aws_project-stamp/aws_project-download-*.log

```



## 吐槽

最后一步失败率极高, 网络不佳的情况 至少得5-10次才能编译完