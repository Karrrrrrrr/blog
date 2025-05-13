---
date: 2025-05-05T12:00:00+08:00
title: K3S
categories: ["工具","容器"]
tags: ["Docker","K3S","Containerd"]
---
# K3S



## 安装k3s

由于网络原因, 国内无法使用官网的链接安装k3s
```bash
curl -sfL https://rancher-mirror.rancher.cn/k3s/k3s-install.sh | INSTALL_K3S_MIRROR=cn sh
```


## k3s无法成功运行pod

遇到如下报错

```
Failed to create pod sandbox: rpc error: code = Unknown desc = failed to get sandbox image "rancher/pause:3.1":
failed to pull image "rancher/pause:3.1": failed to resolve reference "docker.io/rancher/pause:3.1":
dial tcp XXX.XXX.XXX.XXX:443: connect: connection refused
```

### 原因 k3s依赖pause容器, 无法成功从国外拉取pause镜像



1. 编辑 K3s 服务配置：

```shell
sudo vi /etc/systemd/system/k3s.service
```

2. 在 `ExecStart` 中添加：

```shell
--pause-image registry.aliyuncs.com/google_containers/pause:3.1
```

改完之后大概如下
```
[Unit]
Description=Lightweight Kubernetes
Documentation=https://k3s.io
Wants=network-online.target
After=network-online.target

[Install]
WantedBy=multi-user.target

[Service]
Type=notify
EnvironmentFile=-/etc/default/%N
EnvironmentFile=-/etc/sysconfig/%N
EnvironmentFile=-/etc/systemd/system/k3s.service.env
KillMode=process
Delegate=yes
User=root
# Having non-zero Limit*s causes performance problems due to accounting overhead
# in the kernel. We recommend using cgroups to do container-local accounting.
LimitNOFILE=1048576
LimitNPROC=infinity
LimitCORE=infinity
TasksMax=infinity
TimeoutStartSec=0
Restart=always
RestartSec=5s
ExecStartPre=/bin/sh -xc '! /usr/bin/systemctl is-enabled --quiet nm-cloud-setup.service 2>/dev/null'
ExecStartPre=-/sbin/modprobe br_netfilter
ExecStartPre=-/sbin/modprobe overlay
ExecStart=/usr/local/bin/k3s \
    server --pause-image registry.aliyuncs.com/google_containers/pause:3.1\

```


3. 重启 K3s

```shell
sudo systemctl daemon-reload
sudo systemctl restart k3s
```

4. #### **清理并重建 Pod**

```shell
kubectl delete pod <pod-id>
kubectl get pods -w  # 观察新 Pod 状态
```



## k3s无法直接通过自定义端口映射访问pod

使用traefik转发端口

1.  **手动拉取 Traefik 镜像**

```shell
sudo ctr images pull swr.cn-north-4.myhuaweicloud.com/kar/traefik:2.11.5
sudo ctr images tag swr.cn-north-4.myhuaweicloud.com/kar/traefik:2.11.5 docker.io/traefik/traefik:v2.10

# 检查镜像是否存在
sudo ctr images ls | grep traefik
```

### 2. **删除并重新触发 Traefik 安装**

```shell
# 删除卡住的 Helm Job
kubectl delete job -n kube-system helm-install-traefik-crd helm-install-traefik
# 重启 K3s 以重新触发安装
sudo systemctl restart k3s
```

```
helm install traefik -n kube-system \
  --set image.repository=swr.cn-north-4.myhuaweicloud.com/kar/traefik \
  --set image.tag=2.11.5 \
  traefik/traefik
```





## 进入容器

类似docker中的exec命令

```shell
docker exec -it <id/name> /usr/bash
```

在kube中使用如下命令

```shell
kubectl exec -it <pod名称> -- sh
```

## 查看详情

```shell
kubectl describe pod -n kube-system helm-install-traefik-7w94d
```



## 查看日志

```shell
kubectl logs -n kube-system helm-install-traefik-7w94d
```

## 设置源(无效)

```shell
sudo mkdir -p /etc/rancher/k3s
sudo tee /etc/rancher/k3s/registries.yaml <<EOF
mirrors:
  docker.io:
    endpoint:
      - "https://docker.1ms.run" 
      - "https://docker.xuanyuan.me"          # 主要是上面这俩
      - "https://docker.mirrors.ustc.edu.cn"  # 中科大镜像源
      - "https://registry-1.docker.io"         # 备用官方源
EOF
```

https://eut3i7o2.mirror.aliyuncs.com

registry.cn-hangzhou.aliyuncs.com

## 重启k3s服务

```shell
systemctl restart k3s
```

## 查看组件对应的容器依赖

```shell
kubectl get pods -n kube-system -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.spec.containers[0].image}{"\n"}{end}'
```



## 查看服务

```shell
kubectl get pods -n kube-system -w # watch模式
kubectl get pods -n kube-system # 非watch模式
```

```
coredns-ff8999cc5-wrfq8                   0/1     ImagePullBackOff   0          178m
helm-install-traefik-7w94d                0/1     ImagePullBackOff   0          140m
helm-install-traefik-crd-p4zw2            0/1     ImagePullBackOff   0          140m
local-path-provisioner-774c6665dc-t8kc5   0/1     ImagePullBackOff   0          178m
metrics-server-6f4c6675d5-r2nhq           0/1     ImagePullBackOff   0          178m
svclb-nginx-service-626f3b03-bwwcw        0/1     ImagePullBackOff   0          96m

kubectl describe pod -n kube-system coredns-ff8999cc5-wrfq8        				=> rancher/mirrored-coredns-coredns:1.12.0
kubectl describe pod -n kube-system helm-install-traefik-7w94d     				=> rancher/klipper-helm:v0.9.4-build20250113
kubectl describe pod -n kube-system helm-install-traefik-crd-p4zw2  		    => rancher/klipper-helm:v0.9.4-build20250113
kubectl describe pod -n kube-system local-path-provisioner-774c6665dc-t8kc5      => rancher/local-path-provisioner:v0.0.31
kubectl describe pod -n kube-system metrics-server-6f4c6675d5-r2nhq 		    => rancher/mirrored-metrics-server:v0.7.2
kubectl describe pod -n kube-system svclb-nginx-service-626f3b03-bwwcw    		=> rancher/klipper-lb:v0.4.13
kubectl describe pod -n kube-system traefik-67bfb46dcb-8mq9b   			        => rancher/mirrored-library-traefik:3.3.2
```

## 卸载k3s

使用自带的脚本

```
/usr/local/bin/k3s-uninstall.sh
```

## 查看路由分发状态

```
kubectl describe ingress nginx-path-ingress
```

```
Name:             nginx-path-ingress
Labels:           <none>
Namespace:        default
Address:          192.168.88.130
Ingress Class:    <none>
Default backend:  <default>
Rules:
  Host        Path  Backends
  ----        ----  --------
  *
              /api      nginx1-svc:80 (10.42.0.71:80)
              /static   nginx2-svc:80 (10.42.0.72:80)
              /         nginx3-svc:80 (10.42.0.73:80)
Annotations:  nginx.ingress.kubernetes.io/rewrite-target: /
Events:       <none>
```

## 创建服务

```yaml
# nginx-configmaps.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx1-content
data:
  index.html: |
    <html><body><h1>API Server (nginx1)</h1></body></html>
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx2-content
data:
  index.html: |
    <html><body><h1>Static Files Server (nginx2)</h1></body></html>
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx3-content
data:
  index.html: |
    <html><body><h1>Main Server (nginx3)</h1></body></html>
```

```yaml
# nginx-deployments.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx1
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx1
  template:
    metadata:
      labels:
        app: nginx1
    spec:
      containers:
      - name: nginx
        image: swr.cn-north-4.myhuaweicloud.com/kar/nginx:stable-alpine-perl
        ports:
        - containerPort: 80
        volumeMounts:
        - name: nginx-config
          mountPath: /usr/share/nginx/html
      volumes:
      - name: nginx-config
        configMap:
          name: nginx1-content
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx2
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx2
  template:
    metadata:
      labels:
        app: nginx2
    spec:
      containers:
      - name: nginx
        image: swr.cn-north-4.myhuaweicloud.com/kar/nginx:stable-alpine-perl
        ports:
        - containerPort: 80
        volumeMounts:
        - name: nginx-config
          mountPath: /usr/share/nginx/html
      volumes:
      - name: nginx-config
        configMap:
          name: nginx2-content
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx3
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx3
  template:
    metadata:
      labels:
        app: nginx3
    spec:
      containers:
      - name: nginx
        image: swr.cn-north-4.myhuaweicloud.com/kar/nginx:stable-alpine-perl
        ports:
        - containerPort: 80
        volumeMounts:
        - name: nginx-config
          mountPath: /usr/share/nginx/html
      volumes:
      - name: nginx-config
        configMap:
          name: nginx3-content
```

```yaml
# nginx-ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx-path-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - http:
      paths:
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: nginx1-svc
            port:
              number: 80
      - path: /static
        pathType: Prefix
        backend:
          service:
            name: nginx2-svc
            port:
              number: 80
      - path: /
        pathType: Prefix
        backend:
          service:
            name: nginx3-svc
            port:
              number: 80
```

```yaml
# nginx-services.yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx1-svc
spec:
  selector:
    app: nginx1
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: nginx2-svc
spec:
  selector:
    app: nginx2
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: nginx3-svc
spec:
  selector:
    app: nginx3
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```



```shell
kubectl apply -f nginx-configmaps.yaml
kubectl apply -f nginx-deployments.yaml
kubectl apply -f nginx-services.yaml
kubectl apply -f nginx-ingress.yaml
```



## 卸载服务

### 如果原 YAML 文件存在：

```shell
kubectl delete -f nginx-configmaps.yaml
kubectl delete -f nginx-services.yaml
kubectl delete -f nginx-ingress.yaml
kubectl delete -f nginx-deployments.yaml

# 删除目录下的所有部署
kubectl delete -f ./
```

### 如果原 YAML 文件已丢失：

```
# 1. 通过标签删除（如果资源有统一标签）
kubectl delete all -l app=<你的应用标签>

# 2. 删除命名空间内所有资源（慎用！）
kubectl delete all --all -n <命名空间>
```







# Rancher

## Docker 安装Rancher

```shell
docker run -itd -p 8000:80 -p 8443:443  --privileged --restart=unless-stopped -e CATTLE_AGENT_IMAGE="registry.cn-hangzhou.aliyuncs.com/rancher/rancher-agent:v2.11.0" --name rancher registry.cn-hangzhou.aliyuncs.com/rancher/rancher:v2.11.0
```

