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
apt install -y clang cmake libgmp-dev libmpfr-dev libmpc-dev make
```

#### 下载源码
```shell
git clone https://github.com/llvm/llvm-project.git
cd llvm-project
```
 


#### 编译LLVM

```shell
mkdir llvm-build
cd llvm-build
cmake -G "Unix Makefiles"  -DCMAKE_BUILD_TYPE="Release" ../llvm
make -j 4
make install
```

#### 编译Clang

```shell
mkdir clang-build
cd clang-build
cmake -G "Unix Makefiles" -DLLVM_INCLUDE_TESTS=OFF -DCMAKE_BUILD_TYPE="Release" ../clang
make -j 4
make install
```



### 解决方案

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
apt install -y clang cmake libgmp-dev libmpfr-dev libmpc-dev make

git clone https://github.com/llvm/llvm-project.git
cd llvm-project
# 需要一个老版本的C++编译器, g++或者clang++都可以
mkdir llvm-build
cd llvm-build
cmake -G "Unix Makefiles"  -DCMAKE_BUILD_TYPE="Release" ../llvm
# 等待一小时
make -j 4
make install
cd ..
mkdir clang-build
cd clang-build
# 必须要在前面先安装llvm
cmake -G "Unix Makefiles" -DLLVM_INCLUDE_TESTS=OFF -DCMAKE_BUILD_TYPE="Release" ../clang
# 等待两小时
make -j 4
make install

```

