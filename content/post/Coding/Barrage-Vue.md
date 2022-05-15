---
title: "Barrage Vue"
date: 2022-05-15T23:13:05+08:00
---

### 关于Vue播放器弹幕的实现
特点：
- 全屏保持弹幕正常运行， 
- 鼠标经过弹幕会暂停， 
- 双击播放器可以全屏，
- esc或者再次双击退出

```html
<!DOCTYPE html>
<html lang="zh-cn" >
<head >
    <meta charset="UTF-8" >
    <title > Barrage </title >
    <style >
        #container {
            position : relative;
            height   : 800px;
            width    : 1000px;
            overflow : hidden;
            border   : 1px solid black;
        }

        #container video {
            position : absolute;
            height   : 100%;
            width    : 100%;
        }

        #container #wrapper {
            position : absolute;
            z-index  : 100;
            border   : 1px solid black;
            width    : 100%;
            height   : 100%;
        }

        #barrage-box {
            position  : absolute;
            z-index   : 99;
            width     : 200%;
            height    : 100%;
            transform : translateX(-40%);
        }

        .barrage {
            color       : red;
            animation   : barrage 10s linear;
            position    : absolute;
            font-size   : 30px;
            font-family : "DejaVu Math TeX Gyre";
            user-select : none;
        }

        @keyframes barrage {
            from {
                left : 100%;
            }
            to {
                left : -100%;
            }
        }
    </style >
</head >
<body >
<div id="app" >
    <barrage-video />
</div >
</body >
<script src="./vue.global.prod.js" ></script >
<script >
    const {createApp, ref} = Vue


    const useVideo = () => {
        const video = ref(null)
        const barrageList = ref([])
        let id = 0

        function videoClick() {
            video.value.paused ? video.value.play() : video.value.pause()
        }

        function play() {
            video.value.play()
        }

        function pause() {
            video.value.pause()
        }

        function fullScreen() {
            const element = document.querySelector('#container');
            console.log(document.fullscreenEnabled)
            if (document.fullscreenEnabled && document.fullscreenElement == null) {
                element.requestFullscreen()
                return
            }
            document.exitFullscreen()
        }

        function closeFullScreen() {
            document.exitFullscreen()
        }

        function sendBarrage() {
            const top = Math.floor(Math.random() * 30) * 30 + 'px'
            const meta = {
                text: '弹幕', top, time: +new Date(), className: '', id: ++id
            }

            barrageList.value.filter(item => {
                return meta.time - item.time < 10
            })
            barrageList.value.push(meta)
            meta.className = 'barrage'
        }

        function onMouseLeave({target}) {
            target.style.animationPlayState = 'running';
        }

        function onMouseEnter({target}) {
            target.style.animationPlayState = 'paused';
        }

        return {
            video,
            sendBarrage,
            closeFullScreen,
            fullScreen,
            play,
            pause,
            videoClick,
            barrageList,
            onMouseEnter,
            onMouseLeave
        }
    }
    const Video = {
        template: `
          <div id="container" >
          <video src="http://172.22.0.3/video.mp4" ref="video" autoplay ></video >
          <div id="barrage-box" @dblclick="fullScreen" @click="videoClick" >
            <div @mouseenter="onMouseEnter"
                 @mouseleave="onMouseLeave"
                 v-text="item.text"
                 v-for="item in barrageList"
                 :style="{marginTop: item.top}"
                 :class="item.className"
                 :key="item.id" />
          </div >
          </div >
          <button @click="play" >PLAY</button >
          <button @click="pause" >PAUSE</button >
          <button @click="fullScreen" >FullScreen</button >
          <button @click="sendBarrage" >SEND</button >`,
        setup() {
            const video = useVideo()

            return {
                ...video,
            }

        }
    }

    const app = createApp({})
    app.component('BarrageVideo', Video)
    app.mount('#app')

</script >
</html >
```