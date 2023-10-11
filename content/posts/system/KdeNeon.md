---
title: KDE Neon
date: 2022-10-26T02:24:21+08:00 
categories: ["系统" , "Linux", "KDE"]
tags: ["系统" , "Linux" , "KDE"]
--- 

# KDE Neon 22.04

## 应用篇


### 微信

用麒麟的， 不用deepin的， deepin的有bug

麒麟也有bug， 不支持cv进图片， 支持自带的截图工具
deepin版本反之
相比之下 麒麟版本好用



###  Latte-Dock
```shell
git close https://github.com/kde/latte-dock
sudo apt install cmake 
sudo apt install extra-cmake-modules 
sudo apt install qtdeclarative5-dev 
sudo apt install libqt5x11extras5-dev
sudo apt install libkf5iconthemes-dev 
sudo apt install libkf5plasma-dev 
sudo apt install libkf5windowsystem-dev
sudo apt install libkf5declarative-dev
sudo apt install libkf5xmlgui-dev 
sudo apt install libkf5activities-dev
sudo apt install build-essential 
sudo apt install libxcb-util-dev 
sudo apt install libkf5wayland-dev 
sudo apt install git
sudo apt install gettext 
sudo apt install libkf5archive-dev 
sudo apt install libkf5notifications-dev
sudo apt install libxcb-util0-dev
sudo apt install libsm-dev
sudo apt install libkf5crash-dev 
sudo apt install libkf5newstuff-dev
sudo apt install libxcb-shape0-dev
sudo apt install libxcb-randr0-dev 
sudo apt install libx11-dev
sudo apt install libx11-xcb-dev
sudo apt install kirigami2-dev
sudo apt install libwayland-dev
sudo apt install libqt5waylandclient5-dev 
# 好像还有个Waylandprotocol
```
### 某
```text
https://bigairport.date/api/v1/client/subscribe?token=5d1bdb5b98bcaea86041b1f7db353066
https://sub-xsus.com/api/v1/client/subscribe?key=90f1e50f1b70fcae426c785b0939892c
```
### yakuake
```shell
sudo apt install yakuake 
```
### Docker
```shell
# 这里有个坑, 如果是snap安装的docker 默认不创建docker用户, 需要手动创建, apt则不会
sudo gpasswd -a $USER docker 
newgrp docker
```
### zsh
```shell
sudo apt install zsh 
sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
```

```shell
alias cls=clear
alias po=poweroff
alias rb=reboot
export https_proxy=http://localhost:8889
export http_proxy=http://localhost:8889
```

### 网易云音乐
修改文件如下
/opt/netease/netease-cloud-music/netease-cloud-music.bash
```shell
#!/bin/sh
HERE="$(dirname "$(readlink -f "${0}")")"
export LD_LIBRARY_PATH="${HERE}"/libs
export QT_PLUGIN_PATH="${HERE}"/plugins
export QT_QPA_PLATFORM_PLUGIN_PATH="${HERE}"/plugins/platforms
cd /lib/x86_64-linux-gnu/   ## 加这行
exec "${HERE}"/netease-cloud-music $@
```

---

## 系统篇
### 国内源
```text
# deb http://archive.ubuntu.com/ubuntu/ jammy main restricted universe multiverse
# deb http://security.ubuntu.com/ubuntu/ jammy-security main restricted universe multiverse
# deb http://archive.ubuntu.com/ubuntu/ jammy-updates main restricted universe multiverse

# 默认注释了源码镜像以提高 apt update 速度，如有需要可自行取消注释deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-updates main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-updates main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-backports main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-backports main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-security main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-security main restricted universe multiverse
```



### 进不去grub
```shell
set root=()
set prefix=($root)/boot/grub
insmod normal 
normal 
```

### GRUB
更新GRUB无效 不知道为啥
已解决， grub.d文件夹下有theme文件，删了就行
如果update-grub找不到windows分区，用pe修复下就行，原因是update-grub会去efi下找windows

### git
对于不在自己分区的git仓库， 无法直接提交，需要修改权限， 或者忽略权限验证 
如下
```shell
git config --global --add safe.directory "*" 
```

### SSH免密码
```shell
ssh-keygen -t rsa
```
.pub传给服务器
### 主题
```shell
/home/kar/.local/share/aurorae/themes/Lisa/
```


### 开机挂载硬盘
```shell
blkid
# 找到挂载的uuid
sudo vim /etc/fstab
``` 
```text
写入如下，保存即可
UUID="86104B60104B5679"                   /home/kar/Data        ntfs defaults 0 0 
```



### 关于ssh免密码连不上这回事

https://www.cnblogs.com/qyjzjw/p/16173016.html