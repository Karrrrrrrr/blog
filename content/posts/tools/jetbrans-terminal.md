---
title: "Jetbrains Terminal"
date: "2022-08-28T21:13:14+08:00" 
---

Goland的终端分为两种， 一种是过长不换行的，像windows的， 该模式下写gin会被认为是非终端， 日志不会有颜色

另一种是超出屏幕直接截断，换到第二行， 该情况会被认为是终端（pty）， 日志默认有颜色，

切换方法 
修改注册表```go.run.processes.with.pty ```的值, 打钩为pty(强制换行)