---
title: GolangJSON序列化技巧
cover: https://tse3-mm.cn.bing.net/th/id/OIP-C.6WNXKZEe6OrhwI2x-JzRqQHaDt?pid=ImgDet&rs=1
---

### GolangJSON序列化技巧
当我们从数据库里取出数据之后，通常会包装成JSON字符串传给前端。

这里通常有个问题， 假如ID是雪花算法生成的， ID会超过int的范围， 从而在前端丢失进度
因为JS数字最大是1<<53

为了避免这种问题， 最好在传给前端的时候 转化为string

```go
type User struct {
	Id int `json:"Id,string"`
}
```
如上面那样 就能在序列化的时候自动改为字符串

当然也可以手动创建一个VO类， 会比较麻烦

