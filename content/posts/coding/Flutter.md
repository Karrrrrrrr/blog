---
title: "Flutter"
categories: ["代码","Android" ]
tags: ["前端" ,"Android","IOS"]
date: "2022-05-27T00:02:18+08:00"

---

# Flutter
Flutter应用必须包裹在脚手架里， 不然会有奇怪的问题， 比如文字不能正常显示。

## 一些常用参数
### Container 
```text
width: double.infinity 
padding: EdgeInsert.all 
```
container 和 div 不同， div默认撑开width， 但是container需要手动设置。

### MaterialButton 
flutter没有直接的button，只有 MaterialButton 和 FloatButton；

MaterialButton的color属性是用来设置背景色的， 如果要设置文字颜色， 就得设置Text的style为TextStyle（color： xxx）