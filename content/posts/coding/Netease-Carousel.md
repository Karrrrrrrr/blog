---
title: "网易云轮播图"
date: 2022-06-02T21:03:09+08:00

categories: ["代码", "前端", "Vue" ]
tags: ["Vue", "前端" ]
 
---

### 关于 网易云 轮播图的实现

特点：

- 有了图层的概念, 在变化的过程中有了3D的效果

```vue

<template >
  <div id="app" >
    <div class="container" >
      <div :key="item.id" v-for="(item,i) in list"
           class="wrapper" :style="calcStyle(i)" >
        <img :src="item.pic" class="img" @click="onClick(i)" >
      </div >
    </div >
    <div >
      <div class="point-wrapper" >
        <div v-for="(item,i) in list.length" class="point" :style="i==index? {background: 'red'} :{}" ></div >
        <div style="clear: both;" ></div >
      </div >
      <button @click="last" >
        <-last
      </button >
      <span >
        第 {{ index }} 页
      </span >
      <button @click="next" >
        next ->
      </button >
    </div >
  </div >

</template >
```

```javascript

const {createApp, ref, onMounted} = Vue
const useData = () => {
    const list = ref(
        [
            {
                "id": 939543930,
                "pic": "http://i1.hdslb.com/bfs/archive/6e367796be3f715d0802977ef8539bba44bac64d.jpg"
            },
            {
                "id": 769699650,
                "pic": "http://i2.hdslb.com/bfs/archive/b3877be767c8c928c2eacc4d65a45b266f61ec7c.jpg"
            },
            {
                "id": 811854953,
                "pic": "http://i2.hdslb.com/bfs/archive/47c362ba7ec4c9888d350c1c3b20188b578fed4d.jpg"
            },
            {
                "id": 684521845,
                "pic": "http://i1.hdslb.com/bfs/archive/4a5c45e671b78eff144e99bf57b09f055074fd18.jpg"
            },
            {
                "id": 596397229,
                "pic": "http://i2.hdslb.com/bfs/archive/259e84b69650d41dcbeaca0b4fd3339d1833fbdc.jpg"
            },
            {
                "id": 511834767,
                "pic": "http://i2.hdslb.com/bfs/archive/bdace4200dd5d237c6266e8cbbce6c8f5cee3559.jpg"
            },
            {
                "id": 768756943,
                "pic": "http://i1.hdslb.com/bfs/archive/080fcb06db8404f17d2610dd6d677c46bc9cac25.jpg"
            },
            {
                "id": 426909332,
                "pic": "http://i0.hdslb.com/bfs/archive/6b0cb200c302bb5b07417331dfaa5a400a68ce90.jpg"
            },
            {
                "id": 684583815,
                "pic": "http://i2.hdslb.com/bfs/archive/15f0f155d12b32bc07a315c5bb7c19120a8854e3.jpg"
            },
            {
                "id": 342080729, "pic": "http://i1.hdslb.com/bfs/archive/4b2c13a38122aaf2bbacc44bf059cdcf05db4c92.jpg"
            }
        ]
    )
    const N = list.value.length
    return {list, N}
}
const goto = (uri) => {
    window.open(uri)
}
const app = createApp({
    setup() {
        const {list, N} = useData()

        const index = ref(0)

        const next = () => {
            index.value = (index.value + 1) % N
        }
        const last = () => {
            index.value = (index.value - 1 + N) % N
        }

        const getRealPosition = (idx) => {
            return (idx + N) % N
        }

        const calcStyle = (idx) => {
            const scaleValue = 0.6
            const translateX = 70
            const offsetX = 30
            const i = index.value
            if (i === getRealPosition(idx - 2)) {
                return {
                    transform: `translateX(${translateX + offsetX}%) scale(${scaleValue / 1.5})`,
                    zIndex: -2,
                }
            }
            if (i === getRealPosition(idx + 2)) {
                return {
                    transform: `translateX(-${translateX + offsetX}%) scale(${scaleValue / 1.5})`,
                    zIndex: -2,
                }
            }

            if (i === getRealPosition(idx - 1)) {
                return {
                    transform: `translateX(${translateX}%) scale(${scaleValue})`,
                    zIndex: -1,
                }
            }
            if (i === getRealPosition(idx + 1)) {
                return {
                    transform: `translateX(-${translateX}%) scale(${scaleValue})`,
                    zIndex: -1
                }
            }
            if (i === getRealPosition(idx)) {
                return {
                    transform: "translateX(0%)",
                    zIndex: 1
                }
            }
            return {
                zIndex: -3,
                transform: `translateX(${translateX}%) scale(${scaleValue / 2})`,
            }
        }

        const onClick = (i) => {
            index.value = i
        }

        onMounted(() => {
            setInterval(() => {
                next()
            }, 5000)
        })
        return {
            list, goto, index, next, last, getRealPosition, calcStyle, onClick
        }
    }
})


app.mount(document.querySelector('#app'))

```

```scss
#app {
  text-align : center;
  width      : 100vw;
  position   : fixed;
  height     : 100vh;
  left       : 0;
  top        : 0;
  overflow   : auto;
}

.container {
  margin-left : 600px;
  position    : relative;
  height      : 420px;
  width       : 600px;

  .wrapper {
    position      : absolute;
    width         : 600px;
    height        : 400px;
    border-radius : 10px;
    overflow      : hidden;
    margin-top    : 10px;
    transition    : all 600ms;

    .img {
      width      : 100%;
      height     : 100%;
      object-fit : cover;
      cursor     : pointer;

    }
  }


}

.point-wrapper {
  text-align    : center;
  margin-left   : 880px;
  margin-bottom : 20px;

  .point {
    background    : gray;
    width         : 10px;
    height        : 10px;
    float         : left;
    margin-left   : 5px;
    border-radius : 50%;
  }

}

```