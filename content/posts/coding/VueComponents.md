---
title: "VueComponents"
date: "2022-12-29T22:54:10+08:00"
draft: false
---

## 前言

总有一些常用组件，很常见但是网上不一定有， 所以在此记录一些个人认为常用的

##  Outlet 
> 有时会遇到某个组件是全局的(直接渲染在body下面)， 但是又需要局部作用域的一些数据， 此时就可以使用Outlet
```vue

<script >
import { createApp, onMounted, onUnmounted, useSlots } from 'vue'
export default {
  name: 'Outlet',
  setup() {
    const div = document.createElement('div')
    onMounted(() => {
      const slots = useSlots()
      const vn = createApp(slots.default)
      document.body.append(div)
      vn.mount(div)
    })
    onUnmounted(() => {
      div.remove()
    })
    return {}
  }
}
</script >
```

只需在父组件中
```vue
<Outlet>
  <button>xxxxx</button>
</Outlet>
```

> 这个组件好像官方已经实现， Teleport就是


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


## Sprite 
> 雪碧图组件， 这个组件需要依赖图片的尺寸， 局限性比较高

#### 原理
判断鼠标事件即可

```vue
<template >
  <div ref="container"
       class="sprite-container"
  >
    <img :src="props.src"
         class="sprite"
         :style="{transform: `translateX(-${imageOffsetX}%)`, opacity: k===-1? 0: 1}"
         alt="" />
    <div class="process-line"
         :style="{width: `${Math.max(k, 0) * 100}%`}" />
    <div style="height: 100%; width: 100%"
         @mousemove="move"
         @mouseleave="leave" />
  </div >
</template >

<script lang="ts" setup >
import {
  computed,
  ref
} from 'vue'

const container = ref<HTMLDivElement>()
const k = ref(-1)

const props = defineProps<{ src: string, count: number }>()
const imageOffsetX = computed(() => Math.floor(k.value * props.count) * (100 / props.count))

const move = (e: MouseEvent) => {
  if (container.value) {
    const { clientWidth } = container.value
    const { offsetX } = e
    k.value = offsetX / clientWidth
  }
}

const leave = (e: MouseEvent) => {
  k.value = -1
}
</script >

<style >
.process-line {
  border-bottom : 1px solid white;
}

.sprite {
  overflow   : hidden;
  position   : absolute;
  height     : 100%;
  object-fit : cover;
  z-index    : -1;
  opacity    : 0;
  transition : 300ms opacity;
}

.sprite-container {
  overflow : hidden;
  position : relative;
  width    : 100%;
  height   : 100%;
}
</style >
```

生成雪碧图
```shell
fileName=/home/kar/Data/Download/Bilibili/589274595/1/589274595_1_0.mp4
length=$(ffprobe -v error -show_entries format=duration -of default=noprint_wrappers=1:nokey=1 -i ${fileName})
count=20
part=$(echo "$length / $count" | bc)
ffmpeg -v fatal -y -skip_frame nokey -i ${fileName}  -vf "fps=fps=1/${part},scale=200:100,tile=${count}x1" -an -vsync 0 output_%03d.png
```