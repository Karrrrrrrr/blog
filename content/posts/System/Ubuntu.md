---
title: ubuntu-apt 
//cover: https://tse1-mm.cn.bing.net/th/id/OIP-C.GPvt_mGMldIyuljcRHEd5gHaEK?pid=ImgDet&rs=1
cover: https://pic2.zhimg.com/80/v2-616c40f573429e5dafe5b50ac4753329_720w.jpg
categories: ["系统" , "Linux"]

---

把以下代码写入/etc/apt/sources.list
```
deb http://mirrors.163.com/ubuntu/ focal main restricted universe multiverse
deb http://mirrors.163.com/ubuntu/ focal-security main restricted universe multiverse
deb http://mirrors.163.com/ubuntu/ focal-updates main restricted universe multiverse
deb http://mirrors.163.com/ubuntu/ focal-backports main restricted universe multiverse
# deb-src http://mirrors.163.com/ubuntu/ focal main restricted universe multiverse
# deb-src http://mirrors.163.com/ubuntu/ focal-security main restricted universe multiverse
# deb-src http://mirrors.163.com/ubuntu/ focal-updates main restricted universe multiverse
# deb-src http://mirrors.163.com/ubuntu/ focal-backports main restricted universe multiverse
# 预发布软件源，不建议启用
# deb http://mirrors.163.com/ubuntu/ focal-proposed main restricted universe multiverse
# deb-src http://mirrors.163.com/ubuntu/ focal-proposed main restricted universe multiverse
```

然后执行
```shell
apt install update
```
