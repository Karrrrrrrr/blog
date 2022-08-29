---
title: "Encode File"
date: "2022-08-30T00:18:42+08:00"
draft: false
---

```python 
import sys
args = sys.argv
def parse_args():
    print(args)
    if len(args) != 4:
        print('参数有误')
        print('命令行参数为 {-D/-E} { input_filename } { output_filename }')
        return

    if args[1] in ['-D', '--decode']:
        decode(args[2], args[3])
        return
    if args[1] in ['-E', '--encode']:
        encode(args[2], args[3])
        return


def print_help():
    ...


def encode(input_name, dest_name):
    input_file = open(input_name, 'rb')
    out_file = open(dest_name, 'wb')

    out_file.write(b'%PDF-1.4\n')
    out_file.write(input_file.read())
    input_file.close()
    out_file.close()


def decode(input_name, dest_name):
    input_file = open(input_name, 'rb')
    out_file = open(dest_name, 'wb')
    input_file.readline()
    out_file.write(input_file.read())
    input_file.close()
    out_file.close()


parse_args()
```
 
 

给目标文件添加一个文件头， 伪装成pdf

如果是在linux下， 解码只需要以下命令
```shell    
cat input_file | tail -n +2 > out_file
```
目的就是跳过文件头，恢复原本的文件

编码命令如下
```shell
echo  "%PDF-1.4" > out_file && cat infile >> out_file
```