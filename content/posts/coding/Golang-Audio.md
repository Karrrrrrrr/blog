---
title: "Golang-Audio"
date: 2023-02-21T22:26:42+08:00
---

## Golang播放音乐

```go

package main

import (
	"github.com/faiface/beep"
	"github.com/faiface/beep/speaker"
	"github.com/faiface/beep/wav"

	"log"
	"os"
	"time"
)

/*
 使用go语言实现map播放器
*/

func main() {
	// 1. 打开mp3文件
	audioFile, err := os.Open("./周杰伦 - 稻香.wav")
	if err != nil {
		log.Fatal(err)
	}
	// 使用defer防止文件描述服忘记关闭导致资源泄露
	defer audioFile.Close()

	// 对文件进行解码

	audioStreamer, format, err := wav.Decode(audioFile)
	if err != nil {
		log.Fatal(err)
	}

	defer audioStreamer.Close()
	// SampleRate is the number of samples per second. 采样率
	_ = speaker.Init(format.SampleRate, format.SampleRate.N(time.Second/10))

	// 用于数据同步，当播放完毕的时候，回调函数中通过chan通知主goroutine
	done := make(chan bool)
	// 这里播放音乐
	speaker.Play(beep.Seq(audioStreamer, beep.Callback(func() {
		// 播放完成调用回调函数
		done <- true
	})))
	// 等待播放完成
	<-done
}
```