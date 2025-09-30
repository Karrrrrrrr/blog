---
title: "Vmware"
date: "2024-07-01T22:00:00+08:00"
---

# Vmware

## vmdk
当vmware的虚拟机长时间的写入操作之后, 磁盘空间会变大, 随后就算删除数据,vmdk文件大小依然保持不变
进入虚拟机系统中,使用如下命令,减少vmdk占用
```shell
vmware-toolbox-cmd disk shrink /
```

## Vmware DHCP
vmware的ip是由dhcp服务管理
在```C:/ProgramData/VMware```目录中,有两个控制文件
1. ```vmnetdhcp.conf``` 用来配置ip分配的规则
2. ```vmnetdhcp.leases```用来记录虚拟机ip的租约

```
allow unknown-clients;
default-lease-time -1;                # default is 30 minutes
max-lease-time -1;                    # default is 2 hours

# Virtual ethernet segment 1
# Added at 06/12/24 22:28:09
subnet 192.168.190.0 netmask 255.255.255.0 {
range 192.168.190.128 192.168.190.254;            # default allows up to 125 VM's
option broadcast-address 192.168.190.255;
option domain-name-servers 192.168.190.1;
option domain-name "localdomain";
default-lease-time 1800;
max-lease-time 7200;
}
host VMnet1 {
    hardware ethernet 00:50:56:C0:00:01;
    fixed-address 192.168.190.1;
    option domain-name-servers 0.0.0.0;
    option domain-name "";
}
# End

# Virtual ethernet segment 8
# Added at 06/12/24 22:28:09
# 记录IP区间和子网掩码
subnet 192.168.80.0 netmask 255.255.255.0 {
range 192.168.80.128 192.168.80.254;            # default allows up to 125 VM's
option broadcast-address 192.168.80.255;
option domain-name-servers 192.168.80.2;
option domain-name "localdomain";
option netbios-name-servers 192.168.80.2;
option routers 192.168.80.2;
default-lease-time 1800;
max-lease-time 7200;
}
host VMnet8 {
    hardware ethernet 00:50:56:C0:00:08;
    fixed-address 192.168.80.1;
    option domain-name-servers 0.0.0.0;
    option domain-name "";
    option routers 0.0.0.0;
}
# End
```

注意配置如下两行
```
default-lease-time -1;                # default is 30 minutes
max-lease-time -1;                    # default is 2 hours
```
单位是秒, 设置为-1表示永久,修改配置后需要重启dhcp服务生效,或者重启系统



## Ubuntu 22.04+
对于这个版本以及以上的ubuntu, 宿主机开启tun模式会导致dns异常, 
此时手动需修改 ```/etc/systemd/resolved.conf```

```
[Resolve]
# Some examples of DNS servers which may be used for DNS= and FallbackDNS=:
# Cloudflare: 1.1.1.1#cloudflare-dns.com 1.0.0.1#cloudflare-dns.com 2606:4700:4700::1111#cloudflare-dns.com 2606:4700:477
00::1001#cloudflare-dns.com
# Google:     8.8.8.8#dns.google 8.8.4.4#dns.google 2001:4860:4860::8888#dns.google 2001:4860:4860::8844#dns.google
# Quad9:      9.9.9.9#dns.quad9.net 149.112.112.112#dns.quad9.net 2620:fe::fe#dns.quad9.net 2620:fe::9#dns.quad9.net
DNS=8.8.8.8
FallbackDNS=
Domains=
DNSSEC=allow-downgrade  # 这行改为allow-downgrade或者no即可, 在早期版本这里默认是no, 22.04之后默认为yes
#DNSOverTLS=no
#MulticastDNS=no
#LLMNR=no
Cache=yes
#CacheFromLocalhost=no
DNSStubListener=yes
#DNSStubListenerExtra=
#ReadEtcHosts=yes
#ResolveUnicastSingleLabel=no
```

然后执行命令 ```sudo systemctl restart systemd-resolved && ping baidu.com```


有趣的是,这个问题只在```ubuntu```出现, ```debian```, ```opensuse``` 等linux均没有这个问题


