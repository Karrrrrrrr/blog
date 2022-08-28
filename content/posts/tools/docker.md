---
title: "Docker"
cover: "https://images.alphacoders.com/227/thumbbig-227923.webp"
categories: ["工具"]
tags: ["Docker"]


---


Docker指令
```shell
docker ps 
docker images 
docker rm  #(-f) 强制删除
docker rmi #(-f) 强制删除
docker stats 

docker cp from-file to-file // 必须一个是本地 一个是容器 无法跨容器

docker df

docker export 

docker import 
```

## Docker日志限制大小
在docker-compose中配置日志大小， 防止磁盘爆满
```yaml
logging:
  options:
    max-size: '12m'
    max-file: '5'
  driver: json-file
```


## Docker登录认证
```shell
echo '{
  "auths": {
    "registry.cn-hangzhou.aliyuncs.com": {
      "auth": "a2FyMTExMTExOjIwMDEwMjE4MTI3N0R5Xw"
    }
  }
}' >   /home/kar/.docker/config.json
```

本质上就是 Base64("${Username}:${Password}")的返回值

然后存到 /home/kar/.docker/config.json 文件里