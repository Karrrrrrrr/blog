---
title: "Minio"
date: 2024-11-01T15:00:00+08:00

categories: ["后端","Minio"]

tags: ["后端","Minio","对象存储"]
---

## JS直接传文件到minio

首先使用一个后端minio客户端生成一个临时凭证, 用于前端上传,这样确保数据的安全性
然后浏览器通过axios/fetch发送put请求到这个url中,文件即可上传到minio

```go
package main

import (
	"context"
	"fmt"
	"log"
	"net/url"
	"time"

	"github.com/gin-gonic/gin"
	"github.com/google/uuid"
	"github.com/minio/minio-go/v7"
	"github.com/minio/minio-go/v7/pkg/credentials"
)

var (
	client *minio.Client
)

func main() {
	app := gin.Default()

	app.GET("/", func(c *gin.Context) {
		c.File("./index.html")
	})

	app.GET("/pre-signed", func(c *gin.Context) {
		var uid = uuid.New().String()
		var fileName = fmt.Sprintf("%s", uid)
		// 参数配置
		bucketName := "public"     // 存储桶名称
		objectName := fileName     // 文件名称（前端上传时会使用这个名称）
		expiry := time.Minute * 15 // URL 过期时间

		// 生成预签名 URL
		reqParams := make(url.Values)
		reqParams.Set("x-amz-acl", "public-read") // 设置文件的访问权限
		presignedURL, err := client.PresignedPutObject(context.Background(), bucketName, objectName, expiry)
		if err != nil {
			log.Fatalln(err)
		}

		c.JSON(200, gin.H{"url": presignedURL.String()})
	})

	app.Run(":8000")
}

func init() {
	// MinIO 配置
	// 这里不需要写http/https 只需要写ip和端口,注意 这里的端口是minio的api端口, 默认为9000的那个
	endpoint := ""        // MinIO 服务器地址
	accessKeyID := ""     // MinIO Access Key
	secretAccessKey := "" // MinIO Secret Key
	useSSL := true        // 是否使用 SSL, 如果是线上环境有https,就改为true

	// 创建 MinIO 客户端
	minioClient, err := minio.New(endpoint, &minio.Options{
		Creds:  credentials.NewStaticV4(accessKeyID, secretAccessKey, ""),
		Secure: useSSL,
	})
	if err != nil {
		log.Fatalln(err)
	}
	client = minioClient

}

```


```html
<!doctype html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport"
          content="width=device-width, user-scalable=no, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
</head>
<body>
<input type="file" id="fileInput">
<button onclick="send()">
    SEND
</button>
</body>
<script>
    const fileInput = document.querySelector('#fileInput')
    const send = () => {
        const file = fileInput?.files[0]
        if (!file) {
            alert('Please choose a file to input')
            return
        }
        fetch('/pre-signed')
            .then(r => r.json())
            .then((r) => {
                const url = r.url
                if (!url) return Promise.reject()
                return Promise.resolve(url)
            })
            .then(url => {
                return uploadFile(file, url)
            })
            .then(() => {
                alert('done!')
            })
    }

    async function uploadFile(file, url) {
        try {
            const response = await fetch(url, {
                method: 'PUT',
                headers: {
                    'Content-Type': file.type
                },
                body: file
            });

            if (response.ok) {
                console.log("File uploaded successfully!");
            } else {
                console.error("Failed to upload file.");
            }
        } catch (error) {
            console.error("Error uploading file:", error);
        }
    }
</script>
</html>
```

[//]: # (todo)
[//]: # (## 断点传输)

[//]: # ()
[//]: # (## 分片传输)

[//]: # (原理同上, 传输完成之后调用合并接口即可, 注意 这里需要minio.NewCore&#40;&#41;而不是直接minio.New&#40;&#41;)

[//]: # (```go)

[//]: # (// 初始化接口)

[//]: # (minioCore := minio.NewCore&#40;&#41;)

[//]: # (// 申请uploadId)

[//]: # (minioCore.NewMultipartUpload&#40;&#41;)

[//]: # (// 上传单个分片)

[//]: # (minioCore.PutObjectPart&#40;&#41;)

[//]: # (// 已上传列表)

[//]: # (minioCore.ListObjectParts&#40;&#41;)

[//]: # (// 合并分片)

[//]: # (minioCore.CompleteMultipartUpload&#40;&#41;)

[//]: # ()
[//]: # (```)

[//]: # (## 秒传)

