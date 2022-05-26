---
title: "CSS实现正方形展示图片"
date: 2022-05-25T00:52:09+08:00
cover: https://pic4.zhimg.com/80/c3edefc8bdd82043a247b2c05a706527_720w.jpg
categories: ["代码", "前端", "CSS" ]
tags: [ "前端", "CSS" ]
---
### CSS实现正方形展示图片

利用flex布局, 0 0 33.3% 使得每行三个

height对应的是父容器的height,父容器没有height(默认0), 所以不能用height 100%;

但是padding 对应的是父容器的weight,父容器width有值(33.3%), 所以可以用padding-top/bottom: 100%;

如果是正方形, 把height限死在0;

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <style>
        .container {
            display       : flex;
            flex-wrap     : wrap;
            /*align-content : flex-start;*/
            width         : 50%;
        }

        .item {
            border : 1px solid black;
            flex   : 0 0 30%;
            margin : 1%;
        }

        .content {
            padding-bottom : 100%;
            height         : 0;
            width          : 100%;
            background     : #000;
        }

    </style>
</head>
<body>
<div class="container">
    <div class="item">
        <div class="content"></div>
    </div>
    <div class="item">
        <div class="content"></div>
    </div>
    <div class="item">
        <div class="content"></div>
    </div>
    <div class="item">
        <div class="content"></div>
    </div>
    <div class="item">
        <div class="content"></div>
    </div>
    <div class="item">
        <div class="content"></div>
    </div>
    <div class="item">
        <div class="content"></div>
    </div>
    <div class="item">
        <div class="content"></div>
    </div>
</div>
</div>
</body>
</html>
```

