---
title: KDE Neon
date: 2022-05-17T02:24:21+08:00
cover: https://pic2.zhimg.com/80/v2-5163a11deb63b9884a67042455714a89_720w.jpg
categories: ["系统" , "Linux", "KDE"]
tags: ["系统" , "Linux" , "KDE"]
--- 

# KDE Neon 22.04

## 应用篇


### 微信

用麒麟的， 不用deepin的， deepin的有bug



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
```
### 某
```text
https://bigairport.date/api/v1/client/subscribe?token=5d1bdb5b98bcaea86041b1f7db353066
```
### yakuake
```shell
sudo apt install yakuake 
```
### Docker
```shell
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

### 进不去grub
```shell
set root=()
set prefix=($root)/boot/grub
insmod normal 
normal 
```

### GRUB
更新GRUB无效 不知道为啥

### SSH免密码
```shell
ssh-keygen -t rsa
```
.pub传给服务器