---
title: Centos8-yum
---
2021年 Centos8不受支持 以至于yum无法正常工作 
但是rocky8是兼容centos8的
所以可以把centos8的yum源指向rocky8的源
在/etc/yum.repos.d/Centos-Base.repo 里写入 以下代码

```repo
# CentOS-Base.repo
#
# The mirror system uses the connecting IP address of the client and the
# update status of each mirror to pick mirrors that are updated to and
# geographically close to the client.  You should use this for CentOS updates
# unless you are manually picking other mirrors.
#
# If the mirrorlist= does not work for you, as a fall back you can try the 
# remarked out baseurl= line instead.
#
#
 
[base]
name=CentOS-$releasever - Base - mirrors.aliyun.com
failovermethod=priority
baseurl=https://mirrors.aliyun.com/rockylinux/$releasever/BaseOS/$basearch/os/

gpgcheck=1
gpgkey=https://mirrors.aliyun.com/rockylinux/RPM-GPG-KEY-rockyofficial
 
#additional packages that may be useful
[extras]
name=CentOS-$releasever - Extras - mirrors.aliyun.com
failovermethod=priority
baseurl=https://mirrors.aliyun.com/rockylinux/$releasever/extras/$basearch/os/

gpgcheck=1
gpgkey=https://mirrors.aliyun.com/rockylinux/RPM-GPG-KEY-rockyofficial
 
#additional packages that extend functionality of existing packages
[centosplus]
name=CentOS-$releasever - Plus - mirrors.aliyun.com
failovermethod=priority
baseurl=https://mirrors.aliyun.com/rockylinux/$releasever/centosplus/$basearch/os/

gpgcheck=1
enabled=0
gpgkey=https://mirrors.aliyun.com/rockylinux/RPM-GPG-KEY-rockyofficial
 
[PowerTools]
name=CentOS-$releasever - PowerTools - mirrors.aliyun.com
failovermethod=priority
baseurl=https://mirrors.aliyun.com/rockylinux/$releasever/PowerTools/$basearch/os/

gpgcheck=1
enabled=0
gpgkey=https://mirrors.aliyun.com/rockylinux/RPM-GPG-KEY-rockyofficial




```
