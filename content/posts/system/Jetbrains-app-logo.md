---
title: "Jetbrains App && Latte-Dock"
date: 2022-05-17T02:24:21+08:00
cover: https://pic2.zhimg.com/80/v2-5163a11deb63b9884a67042455714a89_720w.jpg
categories: ["系统" , "Linux","Jetbrains","Dock"]
tags: ["系统" , "Linux","Jetbrains","Dock"]

---

关于自定义应用在Latte-Dock中显示不正确的问题
在/usr/share/applications/ 这个目录touch以下文件
```text
# /usr/share/applications/jetbrains-goland.desktop

[Desktop Entry]
Version=1.0
Type=Application
Name=goland
Icon=/opt/Goland/bin/goland.svg
Exec="/opt/Goland/bin/goland.sh" %f
Comment=The smartest Golang IDE
Categories=Development;IDE;
Terminal=false
StartupWMClass=jetbrains-goland


```


```text
# /usr/share/applications/jetbrains-pycharm.desktop

[Desktop Entry]
Version=1.0
Type=Application
Name=Pycharm
Icon=/opt/pycharm/bin/pycharm.svg
Exec="/opt/pycharm/bin/pycharm.sh" %f
Comment=The smartest JavaScript IDE
Categories=Development;IDE;
Terminal=false
StartupWMClass=jetbrains-pycharm
```