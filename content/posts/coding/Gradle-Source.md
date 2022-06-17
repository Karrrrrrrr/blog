---
title: "Gradle-Config"
date: 2022-05-25T00:53:09+08:00
cover: https://tse1-mm.cn.bing.net/th/id/R-C.0469a0fb4e306b776e1c19619aae4682?rik=P9bcz8m7JLb6Zg&riu=http%3a%2f%2flogoju.cn%2fwp-content%2fuploads%2f2019%2f01%2fgradle.jpg&ehk=HyR9Gw3TfdV1mRuDbIVKhyfNAqEMzGkRhJUIV6QQPOU%3d&risl=&pid=ImgRaw&r=0&sres=1&sresct=1
categories: ["代码" , "Android" ,"Gradle"]
tags: ["代码" , "Android" ,"Gradle"]

---
### Gradle国内源参考配置

build.gradle
```gradle
// Top-level build file where you can add configuration options common to all sub-projects/modules.

buildscript {
    
    repositories {
        maven { url 'https://maven.aliyun.com/repository/public' }
        mavenCentral()
        google()
        jcenter()
        maven {url "http://mvn.mob.com/android"}
        maven {url 'http://developer.huawei.com/repo/'}
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:4.1.1'
        

        // NOTE: Do not place your application dependencies here; they belong
        // in the individual module build.gradle files
    }
}

allprojects {
    repositories {
        maven { url 'https://maven.aliyun.com/repository/public' }
        mavenCentral()
        google()
        jcenter()
        maven {url "http://mvn.mob.com/android"}
        maven {url 'http://developer.huawei.com/repo/'}
    }
}

task clean(type: Delete) {
    delete rootProject.buildDir
}

```