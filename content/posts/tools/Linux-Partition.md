---
title: "Linux 分区管理"
date: "2024-07-06T22:00:00+08:00"
---

# Linux分区管理
以下例子通过```vagrant/generic-ubuntu2204```镜像举例
## 扩容分区
当磁盘还有多余的空间,且没有被使用时
使用 ```df -h```命令查看磁盘情况
```
root@ubuntu2204:~# df -h
Filesystem                         Size  Used Avail Use% Mounted on
tmpfs                              198M  1.2M  197M   1% /run
/dev/mapper/ubuntu--vg-ubuntu--lv   62G  4.9G   54G   9% /
tmpfs                              988M     0  988M   0% /dev/shm
tmpfs                              5.0M     0  5.0M   0% /run/lock
/dev/sda2                          2.0G  130M  1.7G   8% /boot
tmpfs                              198M  4.0K  198M   1% /run/user/1000
tmpfs                              198M  4.0K  198M   1% /run/user/0
```
使用 ```lsblk```查看分区大小
```
root@ubuntu2204:~# lsblk
NAME                      MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
loop0                       7:0    0  63.3M  1 loop /snap/core20/1822
loop1                       7:1    0 111.9M  1 loop /snap/lxd/24322
loop2                       7:2    0  49.8M  1 loop /snap/snapd/18357
sda                         8:0    0   128G  0 disk
├─sda1                      8:1    0     1M  0 part
├─sda2                      8:2    0     2G  0 part /boot
└─sda3                      8:3    0   126G  0 part
  └─ubuntu--vg-ubuntu--lv 253:0    0    63G  0 lvm  /
sr0                        11:0    1  1024M  0 rom
```


要将新的剩余空间分配到现有的逻辑卷中，需要执行以下几个步骤：
1. 调整分区表：扩展 sda3 分区以使用新的空间。
2. 通知操作系统新的分区表。
3. 扩展物理卷：使 LVM 识别新的空间。
4. 扩展逻辑卷：将新的空间分配给现有的逻辑卷。
5. 扩展文件系统：使文件系统使用新的空间。


### 调整分区表
   使用 parted 工具调整分区表。以下命令将 sda3 分区调整到 512G：

```shell
sudo parted /dev/sda
(parted) resizepart 3 100%
(parted) quit
```
其中 ```resizepart 3 100%```表示让/dev/sda3 获得所有剩余空间

### 通知操作系统新的分区表
   使用 partprobe 命令通知操作系统新的分区表：

```shell
sudo partprobe
```

### 扩展物理卷
   扩展物理卷以使用 sda3 的新空间：

```shell
sudo pvresize /dev/sda3
```

### 扩展逻辑卷
   扩展逻辑卷 ubuntu-lv 以使用物理卷中的所有剩余空间：
```shell
sudo lvextend -l +100%FREE /dev/ubuntu-vg/ubuntu-lv
```
### 扩展文件系统
   最后，扩展文件系统以使用新的空间：

```shell
sudo resize2fs /dev/ubuntu-vg/ubuntu-lv
```
## 查看结果
再次使用 ```df -h```命令查看磁盘情况
```
root@ubuntu2204:~# df -h
Filesystem                         Size  Used Avail Use% Mounted on
tmpfs                              198M  1.2M  197M   1% /run
/dev/mapper/ubuntu--vg-ubuntu--lv  124G  4.9G  114G   5% /
tmpfs                              988M     0  988M   0% /dev/shm
tmpfs                              5.0M     0  5.0M   0% /run/lock
/dev/sda2                          2.0G  130M  1.7G   8% /boot
tmpfs                              198M  4.0K  198M   1% /run/user/1000
tmpfs                              198M  4.0K  198M   1% /run/user/0
```
再次使用 ```lsblk```查看分区大小
```
root@ubuntu2204:~# lsblk
NAME                      MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
loop0                       7:0    0  63.3M  1 loop /snap/core20/1822
loop1                       7:1    0 111.9M  1 loop /snap/lxd/24322
loop2                       7:2    0  49.8M  1 loop /snap/snapd/18357
sda                         8:0    0   128G  0 disk
├─sda1                      8:1    0     1M  0 part
├─sda2                      8:2    0     2G  0 part /boot
└─sda3                      8:3    0   126G  0 part
  └─ubuntu--vg-ubuntu--lv 253:0    0   126G  0 lvm  /
sr0                        11:0    1  1024M  0 rom
```

执行完这些步骤后，逻辑卷 ubuntu-lv 和它所包含的文件系统应该扩展到新的 128G 空间。
## 关于Alpine
在alpine中,默认没有 
```lvextend```
```pvresize```
```resize2fs```
```parted```
四个命令,需要通过输入以下命令安装
```shell
apk add lvm2 parted e2fsprogs-extra
```


## 总结
总结的命令
以下是依次执行的所有命令：
```shell
sudo parted /dev/sda resizepart 3 100%
sudo partprobe
sudo pvresize /dev/sda3
sudo lvextend -l +100%FREE /dev/ubuntu-vg/ubuntu-lv
sudo resize2fs /dev/ubuntu-vg/ubuntu-lv
```
请确保在执行这些操作之前备份重要数据，以防止任何意外情况。