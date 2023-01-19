---
title: "Webrtc"
date: "2022-01-19T18:21:11+08:00"
draft: false
categories: ["代码", "前端", "Websocket","WebRTC" ]
tags: [ "前端", "Typescript","Websocket" ]
---

```ts
import {ref} from "vue";

export const useMedia = () => {
    const player = ref<HTMLVideoElement>()
    const remotePlayer = ref<HTMLVideoElement>()
    const pc = ref<RTCPeerConnection>()
    const localStream = ref<MediaStream>()
    const ws = ref(new WebSocket('ws://localhost:8080'))
    const state = ref('开始通话')
    const username = (Math.random() + 1).toString(36).substring(7)

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