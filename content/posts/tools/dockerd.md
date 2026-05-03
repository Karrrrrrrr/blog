---
title: "dockerd 相关设置"
date: "2022-08-28T21:00:48+08:00" 
categories: ["工具"]
tags: ["Docker"]
---

## Docker Proxy

``` ~/.docker/config.json```

```json
{
        "proxies": {
                "default": {
                        "httpProxy": "http://192.168.88.1:8889",
                        "httpsProxy": "http://192.168.88.1:8889",
                        "noProxy": "127.0.0.0/8"
                }
        }
}
```
注意 这里的proxies会导致创建后的容器有这几个HTTP_PROXY, HTTPS_PROXY, http_proxy, https_proxy环境变量, 如果不需要的需要手动指定环境变量覆盖这四个, 重启是无效的, 只能重建容器


## Docker Pull 使用代理

如果是常见的镜像，国内厂商里都有缓存， 只需要换源即可

修改
```/etc/docker/daemon.json ```
文件为

```json
{
  "registry-mirrors": [
    "https://docker.mirrors.ustc.edu.cn/",
    "https://eut3i7o2.mirror.aliyuncs.com",
    "https://reg-mirror.qiniu.com"
  ],
}
```

然后执行以下命令重启服务即可

```shell
sudo systemctl daemon-reload
sudo systemctl restart docker
```

换源无法加速自己上传的镜像 那么就需要修改为自己的代理

进行如下操作， 重启服务会导致所有docker容器关闭，记得确认数据的安全性

```shell
sudo mkdir /etc/systemd/system/docker.service.d
# 本质上走https协议， http可以不设置
sudo echo '[Service]
Environment="HTTP_PROXY=http://172.17.0.1:8889/"
Environment="SOCK_PROXY=http://172.17.0.1:1089" 
Environment="HTTPS_PROXY=http://172.17.0.1:8889/"
Environment="NO_PROXY=localhost,127.0.0.1"
' > /etc/systemd/system/docker.service.d/http-proxy.conf  # 这个文件名匹配 *.conf 即可， 只需文件夹名对
# 重启服务
sudo systemctl daemon-reload
sudo systemctl restart docker
```

接下来可以pull走代理了

```shell
docker pull ..... 
```

操作完可能需要改回来 否则失去代理后影响正常运行 只需要删除该文件再次重启即可

```shell
rm /etc/systemd/system/docker.service.d
sudo systemctl daemon-reload
sudo systemctl restart docker
```