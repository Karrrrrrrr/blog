---
title: "Android"
date: 2022-07-06T22:26:42+08:00
---

## 安卓新建项目

### 如果只是想不覆盖原项目
修改build.gradle 文件下面
android.defaultConfig.applicationId的值即可
### 如果是HTML5项目， 需要修改www文件夹的
修改dcloud_control.xml的appid， 然后修改包名， 就不会卡在首屏了