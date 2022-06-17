---
title: "Barrage React"

date: 2022-05-16T00:52:09+08:00 

cover: https://pic4.zhimg.com/80/c3edefc8bdd82043a247b2c05a706527_720w.jpg

categories: ["代码", "前端", "React" ]

tags: ["React", "前端" ]
 
---
### 关于 React 播放器弹幕的实现


特点：
- 全屏保持弹幕正常运行，
- 鼠标经过弹幕会暂停，
- 双击播放器可以全屏，
- esc或者再次双击退出

```typescript jsx
import {MouseEventHandler, useRef, useState} from 'react'
import './App.css'

interface Barrage {
    text: string
    time: number
    id: number
    top: string
    className: string
}

const Video = () => {
    const [id, setId] = useState<number>(0)
    const [barrageList, setBarrageList] = useState<Barrage[]>([])
    const video = useRef<HTMLVideoElement>(null)
    
    
    const videoClick = () => {
        if (video.current)
            video.current.paused ? video.current.play() : video.current.pause()
    }
    const fullScreen = () => {
        if (!video.current) return
        const element = video.current.parentElement as HTMLElement
        if (document.fullscreenEnabled && document.fullscreenElement == null) {
            element.requestFullscreen()
            return
        }
        document.exitFullscreen()
    }


    const onMouseEnter: MouseEventHandler<HTMLDivElement> = (e) => {
        const target: HTMLDivElement = e.target as any as HTMLDivElement
        target.style.animationPlayState = 'paused'

    }
    const onMouseLeave: MouseEventHandler<HTMLDivElement> = (e) => {
        const target: HTMLDivElement = e.target as any as HTMLDivElement
        target.style.animationPlayState = 'running'
    }
    const play = () => {
        if (video.current) video.current.play()
    }
    const pause = () => {
        if (video.current) video.current.pause()
    }
    const sendBarrage = () => {
        const barrage: Barrage = {
            id,
            time: +new Date(),
            text: '弹幕!!!',
            top: Math.floor(Math.random() * 30) * 30 + 'px',
            className: ''
        }
        const list = barrageList.filter(item => {
            return barrage.time - item.time < 10000
        })
        list.push(barrage)
        setBarrageList(list)
        barrage.className = 'barrage'
        setBarrageList(list)
        setId(id + 1)

    }

    const BarrageBox = barrageList.map(item => {
        return <div
            onMouseEnter={onMouseEnter}
            onMouseLeave={onMouseLeave}
            style={{marginTop: item.top}}
            className={item.className}
            key={item.id} >
            {item.text}
        </div >
    })


    return (<div >
        <div id="container" >
            {/*{JSON.stringify(barrageList)}*/}
            <video src="http://172.22.0.3/video.mp4" ref={video} autoPlay={true} />
            <div id="barrage-box"
                 onDoubleClick={fullScreen}
                 onClick={videoClick} >{BarrageBox}</div >
        </div >
        <button onClick={play} > PLAY</button >
        <button onClick={pause} > PAUSE</button >
        <button onClick={fullScreen} > FullScreen</button >
        <button onClick={sendBarrage} > SEND</button >
    </div >)

}

function App() {
    return <Video />
}

export default App


```

```css

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
  animation   : barrage 30s linear;
  position    : absolute;
  font-size   : 30px;
  font-family : "DejaVu Math TeX Gyre";
  user-select : none;
  cursor : pointer;

}

@keyframes barrage {
  from {
    left : 100%;
  }
  to {
    left : -100%;
  }
}
```