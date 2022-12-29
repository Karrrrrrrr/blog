---
title: "Vue3奇技淫巧-Hook篇"
date: "2022-12-29T22:54:10+08:00" 
---

## 前言

总有一些常用hook，很常见但是网上不一定有， 所以在此记录一些个人认为常用的
 

## 文件选择器
> 前端选择文件总是依赖input， 因此我们对input又爱又恨
爱在它的实用性， 恨在它的颜值不高， 常常需要自定义

```typescript
import {
    onMounted,
    onUnmounted
} from 'vue' 
export const useFileSelector = () => {
    const selector: HTMLInputElement = document.createElement('input')

    selector.style.display = 'none'
    selector.type          = 'file'
    const chooseFile       = () => {
        return new Promise((resolve, reject) => {
                const onChange = () => {
                    const file = selector.files?.[0] || null
                    resolve(file)
                    selector.removeEventListener('change', onChange)
                }
                selector.addEventListener('change', onChange)
                selector.click()
            }
        )
    }
    const toURL            = (file: File) => {
        return URL.createObjectURL(new Blob([ file ]))
    }
    onMounted(() => document.body.append(selector))
    onUnmounted(() => selector.remove())

    return {
        chooseFile,
        toURL
    }
}
```

> 使用如上代码， 暴露一个chooseFile函数，
> 调用chooseFile函数，
> 等待异步返回的结果就是用户选择的文件

> 这里需要注意，如果用户没有选择文件，
> 而是关闭的对话框，
> 会返回null，所以需要特判

