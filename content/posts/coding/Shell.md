---
title: "Linux Shell Command"
date: 2024-11-01T12:00:00+08:00
---


# Linux命令快速查询
## 基本命令
### awk 


### ps

```shell
# 通常和grep配合使用
ps -elf | grep ...

# eg. 寻找python的相关进程 
ps -elf | grep python 
# 搜索到进程行,然后提取出 pid
# $4 表示pid在第四列(从1开始计数) 
# 为什么需要grep -v grep? 
# 当我们使用grep python作为过滤条件, grep python本身也会生成一个进程, 这样会干扰搜索条件
ps -elf | grep python | grep -v grep | awk '{print $4}'

# 提取pid存入变量, 然后做一些其他事情
pid=$(ps -elf | grep python | grep -v grep | awk '{print $4}' | head -n 1)
echo "$pid" # 这里也可以直接kill掉这个进程

```
### pgrep
```shell
# 用来搜索进程, 同上,更简单
pgrep python 
```

```shell
# 循环处理里面的所有结果
for pid in $(pgrep python); do
    echo "Processing PID: $pid"
    # 在这里可以对每个 PID 执行单独的处理，比如杀死进程
    kill "$pid"
done

```

### du
```shell
# 列出某目录下所有文件夹的大小(包括子文件夹)
du -h 

# 列出某目录下所有文件夹的大小(包括子文件夹),深度为1
du -h -d 1 
```
### top 
```shell
top 
top -1 
```

### df
```shell
# 列出文件系统的占用大小比例
df 
```
### echo

### bash
以下解释出自ChatGPT

```text
在 Bash 中，单引号和双引号的作用不同，主要体现在转义处理和变量解析上：
单引号（'）：表示“严格字面量”，即其中的所有字符都不会被解析或转义。无论是 $、\、" 等特殊字符，在单引号内都不会生效，因此在单引号中使用 \' 是没有意义的，Bash 不支持这种转义方式。

双引号（"）：则允许变量和部分转义字符的解析，如 \"、\$、\ 等。因此双引号更灵活，可以在其中使用 \" 这种方式来插入双引号。

为什么不改成允许单引号转义？
Bash 选择这种设计是为了提供严格的文本引用。单引号的严格性对避免变量意外解析和确保字面意义的文本传递非常有帮助。允许单引号中使用转义字符会降低它的“字面性”，也会让引号系统变得更复杂。

如何简化单引号的嵌套问题？
如果觉得单引号嵌套过于复杂，以下方式可以简化 Bash 脚本中的这种场景：

优先使用双引号，因为双引号内可以随意使用单引号：bash -c "echo 'hello world'"。

使用 printf 替代 echo，它对包含复杂字符的字符串处理更灵活：
```
```shell
# 后面的字符串中需要处理转义
bash -c "echo \"hello world\""
# 如果是单引号,需要特殊处理 '\''=> '
 
bash -c 'echo '\''hello world'\'''
```

### find
```shell
find [路径] [匹配条件] [动作]
# eg.
# 搜索所有后缀为.go的文件
# find ./ -name "*.go" 
```
###

### mount
这个命令使用频率很低, 用来挂载一些镜像

通过下面的例子可以制作一个文件系统镜像

```shell
if [ "$(id -u)" -ne 0 ]; then
    echo "This script must be run as root." >&2
    exit 1
fi
ehco $user;
# rm -rf alpine-rootfs.img
# 解压缩 alpine rootfs 文件
# Create a temp directory to save unzip file
mkdir -p ./mnt/alpine
# to unzip
tar -xzf alpine-rootfs.tar.gz -C ./mnt/alpine

# 获取 rootfs 大小（以 MB 为单位）
rootfs_size=$(du -sm ./mnt/alpine | cut -f1)

# 增加一些额外的空间以确保 ext4 格式化后有足够空间
buffer_size=$((rootfs_size + rootfs_size/4))  # 增加 25% 空间

# 创建 .img 文件，大小为 rootfs 大小加 buffer
dd if=/dev/zero of=alpine-rootfs.img bs=1M count=$buffer_size

# 格式化镜像文件为 ext4 文件系统
mkfs.ext4 alpine-rootfs.img

# 挂载镜像文件
mkdir -p ./mnt/alpine-img
# this command must have root permission, so we must add prefix with "sudo"
mount -o loop alpine-rootfs.img ./mnt/alpine-img

# 将解压的文件复制到挂载的镜像
rsync -a ./mnt/alpine/ ./mnt/alpine-img/

# 卸载镜像
umount ./mnt/alpine-img
# remove temp directory
rm -rf ./mnt/alpine

echo "done!"

```


### grep
搜索目录下文件内容包含的行

比如 搜索golang源码下包含sync.Once的文件行
```shell
grep -nr "sync\.Once" "$(dirname $(which go))/../src"
```
同上 然后统计数量
```shell
grep -nr "sync\.Once" "$(dirname $(which go))/../src" | wc -l
```
# 拓展命令
## docker 

```shell
docker ps 
```

```shell
docker images 
```

```shell
docker ps -a 
```

```shell
docker rm  
# 如果是前缀 必须是唯一前缀
# 或者完整id
```

```shell
docker kill {container_id/container_name_prefix }
# 如果是前缀 必须是唯一前缀
# 或者完整id
```
```shell
docker search {key}
# docker search golang 
```
```shell
docker cp {from} {to}
# eg.
# docker cp ./xxx.txt alpine:/xxx.txt 
```
## rsync 

## rclone 
