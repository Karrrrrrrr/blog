---
title: "Vueuse"

date: "2022-07-30T17:09:37+08:00" 

---

官网 : https://vueuse.org/



### 源码解析

### useLocalstorage
这个实现响应式的原理是 window下面有事件storage 代码如下
```js
window.addEventListener('storage', ()=>{
    // do any thing 
})
```

#### useEvent
dom下面可以自己定义事件 代码如下
```js
const el = document.createElement('div')
el.addEventListener('xxxx')
el.dispatchEvent(new CustomEvent('xxxx'))
```


#### useMutationObserver / ResizeObserver
js提供了MutationObserver 可以监听一些东西
ResizeObserver监听dom.size变化, 原生代码如下
```js
new MutationObserver()
new ResizeObserver()
```
