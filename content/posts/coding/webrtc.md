---
title: "Webrtc"
date: "2022-01-19T18:21:11+08:00"
draft: false
categories: ["代码", "前端", "Websocket","WebRTC" ]
tags: [ "前端", "Typescript","Websocket" ]
---


# Webrtc

### hooks/use-rtc.ts    
```ts
import {onMounted, ref} from "vue";

export const useMedia = () => {
    const player = ref<HTMLVideoElement>()
    const remotePlayer = ref<HTMLVideoElement>()
    const pc = ref<RTCPeerConnection>()
    const localStream = ref<MediaStream>()
    const ws = ref()
    const state = ref('开始通话')
    const username = (Math.random() + 1).toString(36).substring(7)
    onMounted(() => {
        if (!ws.value)
            ws.value = new WebSocket('ws://localhost:8080/ws')

    })

    const open = () => {
        initWs()
        getMediaDevices().then(() => {
            createRtcConnection()
            addLocalStreamToRTC()
        })
    }
    const initWs = () => {
        // ws.value.onopen = () => console.log('ws 已经打开')
        ws.value.onmessage = wsOnMessage
    }

    const wsOnMessage = (e: MessageEvent) => {
        const wsData = JSON.parse(e.data)
        // console.log('wsData', wsData)
        const wsUsername = wsData['username']
        // console.log('wsUsername', wsUsername)
        if (username === wsUsername) return
        const wsType = wsData['type']
        if (wsType === 'offer') {
            const wsOffer = wsData['data']
            pc.value?.setRemoteDescription(new RTCSessionDescription(JSON.parse(wsOffer)))
            state.value = '请接听通话'
        }
        if (wsType === 'answer') {
            const wsAnswer = wsData['data']
            pc.value?.setRemoteDescription(new RTCSessionDescription(JSON.parse(wsAnswer)))
            state.value = '通话中'
        }
        if (wsType === 'candidate') {
            const wsCandidate = JSON.parse(wsData['data'])
            pc.value?.addIceCandidate(new RTCIceCandidate(wsCandidate))

        }
    }
    const sendMessage = (type: string, data: any) => {
        ws.value.send(JSON.stringify({
            username,
            type,
            data,
        }))
    }
    const getMediaDevices = async () => {
        const stream = await navigator.mediaDevices.getUserMedia({video: true, audio: false})
        localStream.value = stream
        player.value!.srcObject = stream
    }
    const createRtcConnection = () => {
        const _pc = new RTCPeerConnection({
            iceServers: [{urls: ['stun:stun.stunprotocol.org:3478']}]
        })
        _pc.onicecandidate = ({candidate}) => {
            if (candidate) sendMessage('candidate', JSON.stringify(candidate))
        }

        _pc.ontrack = e => {
            remotePlayer.value!.srcObject = e.streams[0]
            console.log(remotePlayer.value!.srcObject)
        }

        pc.value = _pc
    }

    const createOffer = async () => {
        const sdp = await pc.value?.createOffer({
            offerToReceiveAudio: true,
            offerToReceiveVideo: true
        })
        pc.value?.setLocalDescription(sdp)
        sendMessage('offer', JSON.stringify(sdp))
        state.value = '等待对方接听'

    }

    const createAnswer = async () => {
        const sdp = await pc.value?.createAnswer({
            offerToReceiveVideo: true,
            offerToReceiveAudio: true,
        })
        pc.value?.setLocalDescription(sdp)
        sendMessage('answer', JSON.stringify(sdp))
        state.value = '通话中'
    }
    const addLocalStreamToRTC = () => {
        localStream
            .value
            ?.getTracks()
            .forEach(track => {
                pc.value?.addTrack(track, localStream.value!)
            })
    }
    return {
        player,
        getMediaDevices,
        createOffer,
        remotePlayer,
        createAnswer,
        state,
        open
    }
}

```

### App.vue

```vue

<script setup lang="ts">
import {useMedia} from "./hooks/use-rtc";
import {onMounted} from "vue";

const {remotePlayer, player, open, state, createOffer, createAnswer, getMediaDevices} = useMedia()
onMounted(() => {
  open()
})
</script>

<template>
  <div>
    {{ state }}
  </div>
  <video ref="remotePlayer" class="player" controls autoplay></video>
  <video ref="player" class="player" controls autoplay></video>
  <div>
    <button @click="createOffer"> create offer</button>
  </div>
  <div>
    <button @click="createAnswer"> create ans</button>
  </div>
</template>

<style scoped>
.player {
  width: 500px;
  height: 300px;
  outline: 1px solid black;
}
</style>

```


### app.go 

```go
package main

import (
	"github.com/gin-gonic/gin"
	"s/ws"
)

func main() {
	var app = gin.Default()

	ws.InitWs(app)

	app.Run(":8080")

}


```

### ws/ws.go

```go
package ws

import (
	"fmt"
	"github.com/gin-gonic/gin"
	"github.com/gorilla/websocket"
	"log"
	"net/http"
)

type Room struct {
	RoomId string `json:"roomId,omitempty"`
}
type ConnectPool map[*websocket.Conn]bool

var mp = ConnectPool{}

var upgrader = websocket.Upgrader{
	// 解决跨域问题
	CheckOrigin: func(r *http.Request) bool {
		return true
	},
} // use default options

func ws(c *gin.Context) {
	ws, err := upgrader.Upgrade(c.Writer, c.Request, nil)
	if err != nil {
		return
	}
	mp[ws] = true
	defer func() {
		ws.Close()
		delete(mp, ws)
	}()
	for {
		mt, message, err := ws.ReadMessage()
		fmt.Println(string(message))
		if err != nil {
			log.Println("write:", err)
			break
		}
		for conn := range mp {
			if conn == ws {
				continue
			}
			conn.WriteMessage(mt, message)
		}
	}
}

func InitWs(app *gin.Engine) {
	app.GET("/ws", ws)
}


```
