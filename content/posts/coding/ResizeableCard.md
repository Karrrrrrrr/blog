---
title: "ResizeableCard"
date: 2025-05-04T18:00:00+08:00
---
# ResizeableCard
灵感来自 <a href="https://chat-preview.lobehub.com/chat">lobehub</a>
![img.png](/template/img/resizeable.png)
图中红色部分边框是可拖动的区域
## Vue 实现

```html
<script setup lang="ts">
import {
  computed,
  ref,
  type StyleValue
} from "vue";

const emit = defineEmits([
  'update:currentWidth',
  'update:currentHeight'
])

const props = defineProps({
  left: {
    type: Boolean,
    default: false
  },
  right: {
    type: Boolean,
    default: false
  },
  bottom: {
    type: Boolean,
    default: false
  },
  top: {
    type: Boolean,
    default: false
  },


  initialWidth: {
    type: [Number, String],
    default: '100%'
  },
  initialHeight: {
    type: [Number, String],
    default: '100%'
  },

  borderColor: {
    type: String,
    default: 'rgba(0,0,0,.18)'
  },
  hoverBorderColor: {
    type: String,
    default: 'rgba(0,0,0,.8)'
  },
  borderZIndex: {
    type: Number,
    default: 100
  }

})

// 不需要给pos套ref, 避免不必要的监听开销
let pos = { screenX: 0, screenY: 0 }
const classList = ['on-drag'] as string[]
const handleDragBegin = ({ screenX, screenY }: MouseEvent, dx: number, dy: number) => {
  const fn = (e: MouseEvent) => handleDrag(e, dx, dy)

  pos = { screenX, screenY }
  classList.forEach(name => {
    document.body.classList.add(name)
  })

  window.addEventListener('mousemove', fn)
  window.addEventListener('mouseup', () => {
    window.removeEventListener('mousemove', fn)
    handleDragEnd()
  })

}
const handleDrag = (e: MouseEvent, dx: number, dy: number) => {
  const { screenX, screenY } = e
  if (screenY && screenX) {
    dragOffsetX.value = (screenX - pos.screenX) * dx / devicePixelRatio
    dragOffsetY.value = (screenY - pos.screenY) * dy / devicePixelRatio
  }
}
const handleDragEnd = () => {
  // commit this change
  currentWidth.value += dragOffsetX.value
  currentHeight.value += dragOffsetY.value

  emit('update:currentWidth', currentWidth.value)
  emit('update:currentHeight', currentHeight.value)

  // clear tmp data
  dragOffsetX.value = 0
  dragOffsetY.value = 0

  // reset style
  classList.forEach(name => {
    document.body.classList.remove(name)
  })
}

const currentWidth = ref(0)
const currentHeight = ref(0)
const dragOffsetX = ref(0)
const dragOffsetY = ref(0)

const toLen = (x: string | number, unit = 'px') => {
  if (typeof x === 'number') return x + unit
  return x
}

const width = computed(() => `calc(${toLen(props.initialWidth)} + ${toLen(currentWidth.value)} + ${toLen(dragOffsetX.value)})`)
const height = computed(() => `calc(${toLen(props.initialHeight)} + ${toLen(currentHeight.value)} + ${toLen(dragOffsetY.value)})`)
const containerStyle = computed(() => {
  return {
    position: 'relative',
    height: height.value,
    width: width.value,
    '--border-color': props.borderColor,
    '--hover-border-color': props.hoverBorderColor,
    '--border-z-index': props.borderZIndex,
  } as StyleValue
})
</script>

<template>

  <div :style="containerStyle">
    <div @mousedown="handleDragBegin($event, 0, -1)"
         v-if="top"
         class="top">
      <div class="inner"/>
    </div>
    <div @mousedown="handleDragBegin($event, 0, 1)"
         v-if="bottom"
         class="bottom">
      <div class="inner"/>
    </div>
    <div @mousedown="handleDragBegin($event, -1, 0)"
         v-if="left"
         class="left">
      <div class="inner"/>
    </div>
    <div @mousedown="handleDragBegin($event, 1, 0)"
         v-if="right"
         class="right">
      <div class="inner"/>
    </div>
    <slot/>
  </div>
</template>

<style lang="scss" scoped>

.var {
  --border-z-index: '';
  --border-color: '';
  --hover-border-color: '';
}

.top, .bottom, .right, .left {
  --line-weight: 10px;
  --line-weight-rate: 1px;
  position: absolute;
  transition: all 100ms;
  display: flex;
  align-items: center;
  justify-content: center;
  //background: var(--border-color);
  z-index: var(--border-z-index);
}

.top:hover .inner,
.bottom:hover .inner,
.right:hover .inner,
.left:hover .inner {
  background: var(--hover-border-color, var(--border-color));
}

// 鼠标悬浮后增大显示面积, 但是不修改实际的面积
// 以父容器大小作为拖拽的碰撞面积, 子组件作为显示的面积, 优化使用体验
.top:hover,
.bottom:hover,
.right:hover,
.left:hover {
  --line-weight-rate: 1px;

}

.right, .left {
  cursor: col-resize;
  height: 100%;
  width: var(--line-weight)
}

.left {
  transform: translateX(-50%);
  left: 0;
}

.right {
  transform: translateX(50%);
  right: 0;
}

.top {
  transform: translateY(-50%);
  top: 0;
}

.bottom {
  transform: translateY(50%);
  bottom: 0;
}

.top, .bottom {
  cursor: row-resize;
  width: 100%;
  height: var(--line-weight)
}

.right .inner,
.left .inner {
  background: var(--border-color);
  height: 100%;
  width: var(--line-weight-rate);
}

.top .inner,
.bottom .inner {
  background: var(--border-color);
  width: 100%;
  height: var(--line-weight-rate);
}

.top .inner,
.bottom .inner,
.right .inner,
.left .inner {
  transition: all 300ms;
}
</style>

<style>
.on-drag {
  user-select: none;
}
</style>

```


## React实现

```tsx
// todo 
```