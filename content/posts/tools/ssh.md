---
title: "Ssh"
date: "2022-08-28T21:21:07+08:00" 
---

一般ssh一分钟不操作就会掉线， 导致卡住终端
```shell
sudo vim /etc/ssh/ssh_config
```
修改为
```
ServerAliveInterval 30  # 每隔多少时间发送一次心跳 (30s)
ServerAliveCountMax 999 # 多少次未响应之后掉线   
```


或者直接修改目标服务器的配置
```shell
# 打开
sudo vim/etc/ssh/sshd_config
```
修改为
```
# 添加
ClientAliveInterval 30 # 服务器30s向客户端发送一次
ClientAliveCountMax 6 
```
