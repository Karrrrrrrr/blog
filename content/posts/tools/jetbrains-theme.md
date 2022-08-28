---
title: "Jetbrains Theme"
date: "2022-07-12T22:49:39+08:00"
categories: ["工具"]
tags: ["Jetbrains"]
---

# 修改jetbrains系App启动图片
看旧了idea的老图片总想换换图片,养眼,而且新版本图片实在不好看

先确认一下要修改的目标文件, 都在 ```./lib``` 目录下
1. idea （platform-impl.jar, 老版本可能是resource.jar）
2. pycharm （pycharm.jar）
3. goland (golang.jar)
4. clion（clion.jar, 这个改了没用, 启动速度太快了）
5. 其他App, 大概率是App名, 新版本可能同一改名叫app.jar了。 

新版本好像都在 ```app.jar```

确定了jar的位置之后,用压缩文件打开jar, 找到壁纸的位置与名字
找一张png的图片（别的格式不行,得用ffmpeg转下格式）, 转完要与原壁纸同名
> 修改之前记得备份jar, 防止改崩
```shell
ffmpeg -i in.jpg  -vf scale=640:-1  out.png
```
转完格式之后, 用以下命令更新目标jar
```shell
jar -uvf $jarName $imgName
```
