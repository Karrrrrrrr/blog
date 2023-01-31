---
title: "Flusher"
date: "2023-01-31T20:07:11+08:00"
draft: false
---

```go
package main

import (
	"fmt"
	"log"
	"net/http"
	"time"
)

func main() {
	http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		w.Header().Set("x-request-id", "x-request-id")

		f, _ := w.(http.Flusher)

		for i := 0; i < 10; i++ {
			// 此处改成文件读取和写入即可实现流式下载
			fmt.Fprintf(w, "time.now(): %v \r\n", time.Now())
			f.Flush()
			time.Sleep(time.Second)
		}

	})

	log.Fatal(http.ListenAndServe(":8888", nil))
}

```


```python
import requests


def get_stream(url):
    s = requests.Session()

    # with s.get(url, headers=None, stream=True) as resp:
    resp = requests.get(url, stream=1)
    # resp.iter_lines()
    x = 1 << 8
    # 每次读取x个字节
    for chunk in resp.iter_content(x):
        if chunk:
            print(chunk)
    resp.close()


url = 'http://127.0.0.1:8888'
get_stream(url)

```