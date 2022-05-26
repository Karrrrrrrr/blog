---
title: Golang Websocket
cover: https://tse3-mm.cn.bing.net/th/id/OIP-C.6WNXKZEe6OrhwI2x-JzRqQHaDt?pid=ImgDet&rs=1
date: 2022-05-08T18:56:46+08:00
categories: ["代码", "Golang" ]
tags: [ "Golang" ,"Websocket"]

---

### Golang Gin Websocket



```go
package ws

import (
	"encoding/json"
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

var ConnectMap = map[string]ConnectPool{}

var upgrader = websocket.Upgrader{
	// 解决跨域问题
	CheckOrigin: func(r *http.Request) bool {
		return true
	},
} // use default options

func ws(c *gin.Context) {
	ws, err := upgrader.Upgrade(c.Writer, c.Request, nil)
	var roomName = ""
	if err != nil {
		//log.Print("upgrade:", err)
		return
	}
	defer func() {
		ws.Close()
		delete(ConnectMap[roomName], ws)
	}()
	for {
		mt, message, err := ws.ReadMessage()
		if roomName == "" {
			room := Room{}
			json.Unmarshal(message, &room)
			roomName = room.RoomId
			fmt.Println(ConnectMap)
			pool := ConnectMap[roomName]
			if pool==nil {
				ConnectMap[roomName] = map[*websocket.Conn]bool{}
			}
			ConnectMap[roomName][ws] = true
		}
		if err != nil {
			log.Println("read:", err)
			break
		}
		log.Printf("recv: %s", message)
		err = ws.WriteMessage(mt, message)
		if err != nil {
			log.Println("write:", err)
			break
		}
	}
}

func InitWs(app *gin.Engine) {
	app.GET("/ws", ws)
}
```