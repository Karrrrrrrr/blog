---
title: "Alipay"
date: 2025-05-01T22:00:00+08:00
---
# 对接支付宝

## return-url
这个参数相对不是很重要, 在浏览器支付完成后会重定向的地址, 如果客户提前关了页面, 那么等于配置无效

## notify-url
这个参数用于接受支付完成后的通知, 如果收不到, 那就只能通过轮询的方式, 效率低下

这个地址如果是https协议, 那么必须是安全的https

验证方式: 直接从浏览器打开这个链接, 如果浏览器提示不安全 那么就是收不到回调的

方式一: 直接使用服务器配置ssl证书 (可以使用traefik自动获取ssl证书)

方式二: 使用内网穿透并配置ssl证书 (推荐使用 https://www.natfrp.com/, 配置方式参照 https://doc.natfrp.com/frpc/auto-https.html)

注意:不要使用ip, 必须是frp分配的域名才有效


## TradePagePay
该方法返回一个网页支付的url, 推荐适用于桌面端网页开发

```go
func TradePagePay(money int, subject string, outTradeNo uint64) (string, error) {
	client, err := GetClient()
	//请求参数
	bm := make(gopay.BodyMap)
	// 这是必填参数, 其他的看情况填
	bm.Set("subject", subject)
	bm.Set("out_trade_no", outTradeNo)
	bm.Set("total_amount", fmt.Sprintf("%.2f", float64(money)/100.0))
	bm.Set("product_code", "FAST_INSTANT_TRADE_PAY")
	//电脑网站支付请求
	payUrl, err := client.TradePagePay(ctx, bm)
	if err != nil {
		log.Println("err:", err)
		return "", err
	}
	return payUrl, err
}
```

## TradeWapPay
该方法返回一个网页支付的url, 推荐适用于移动端网页开发(能自动唤醒支付宝支付)

```go
func TradeWapPay(money int, subject string, outTradeNo uint64) (string, error) {
	client, err := GetClient()
	//请求参数
	bm := make(gopay.BodyMap)
	bm.Set("subject", subject)
	bm.Set("out_trade_no", outTradeNo)
	bm.Set("total_amount", fmt.Sprintf("%.2f", float64(money)/100.0))
	bm.Set("product_code", "FAST_INSTANT_TRADE_PAY")
	//移动端网站支付请求
	payUrl, err := client.TradeWapPay(ctx, bm)
	if err != nil {
		log.Println("err:", err)
		return "", err
	}
	return payUrl, err
}
```

## 唤醒支付宝原理
发起 alipays协议的请求, 类似如下代码

```js
location.href = 'alipays://platformapi/startApp?appId=20000125' + params
```

注意appId是20000125, 而不是网上说的10000007或者20000067

10000007或者20000067都会先进入支付宝, 然后跳转到一个网页, 再次跳回来, 这是错误的

下面是一些常用的唤醒支付宝某项功能的命令
```text
mes = “alipayqr://platformapi/startapp?saId=10000003”;手机充值
mes = “alipayqr://platformapi/startapp?saId=10000007”;扫一扫
mes = “alipayqr://platformapi/startapp?saId=10000009”;爱心捐赠
mes = “alipayqr://platformapi/startapp?saId=100000011”;彩票首页
mes = “alipayqr://platformapi/startapp?saId=100000033”;话费卡转让
mes = “alipayqr://platformapi/startapp?saId=10000110”;关于
mes = “alipayqr://platformapi/startapp?saId=10000112”;服务授权
mes = “alipayqr://platformapi/startapp?saId=20000116”;转账
mes = “alipayqr://platformapi/startapp?saId=20000056”;付款码
mes = alipays://platformapi/startapp?appId=20000003;账单
mes = alipays://platformapi/startapp?appId=20000076;账单
mes = “alipays://platformapi/startapp?appId=20000006”;切换账户
mes = “alipays://platformapi/startapp?appId=20000008”;支付宝登出
mes = “alipays://platformapi/startapp?appId=20000009”;手机号注册
mes = “alipays://platformapi/startapp?appId=20000010”;绑定手机
mes = “alipays://platformapi/startapp?appId=20000011”;客服帮助
mes = “alipays://platformapi/startapp?appId=20000013”;修改支付密码
mes = “alipays://platformapi/startapp?appId=20000014”;我的银行卡
mes = “alipays://platformapi/startapp?appId=20000015”;找回登录密码
mes = “alipays://platformapi/startapp?appId=20000017”;修改登录密码
mes = “alipays://platformapi/startapp?appId=20000019”;余额
mes = “alipays://platformapi/startapp?appId=20000020”;卡包
mes = “alipays://platformapi/startapp?appId=20000024”;支付宝设置
mes = “alipays://platformapi/startapp?appId=20000027”;账号切换
mes = “alipays://platformapi/startapp?appId=20000031”;设置个人头像
mes = “alipays://platformapi/startapp?appId=20000032”;余额宝
mes = “alipays://platformapi/startapp?appId=20000033”;提现
mes = “alipays://platformapi/startapp?appId=20000038”;身份验证
mes = “alipays://platformapi/startapp?appId=20000048”;添加生活好
mes = “alipays://platformapi/startapp?appId=20000049”;意见反馈
mes = “alipays://platformapi/startapp?appId=20000050”;打开地图
mes = “alipays://platformapi/startapp?appId=20000057”;账号管理
mes = “alipays://platformapi/startapp?appId=20000068”;快速挂失
mes = “alipays://platformapi/startapp?appId=20000068”;安全中心
mes = “alipays://platformapi/startapp?appId=20000071”;城市一卡通
mes = “alipays://platformapi/startapp?appId=20000078”;上银汇款
mes = “alipays://platformapi/startapp?appId=20000081”;更多
mes = “alipays://platformapi/startapp?appId=20000122”;首页活动
mes = “alipays://platformapi/startapp?appId=20000123”;收钱
mes = “alipayqr://platformapi/startapp?appId=68687017”;年度账单
mes = “alipayqr://platformapi/startapp?appId=20000101”;生活号
mes = “alipayqr://platformapi/startapp?appId=20000102”;打开nfc
mes = “alipayqr://platformapi/startapp?appId=20000107”;出境
mes = “alipayqr://platformapi/startapp?appId=20000108”;挂号就诊
mes = “alipayqr://platformapi/startapp?appId=20000110”;我的保障
mes = “alipayqr://platformapi/startapp?appId=20000115”;设备管理
mes = “alipayqr://platformapi/startapp?appId=20000119”;阿里游戏
mes = “alipayqr://platformapi/startapp?appId=20000118”;芝麻信用
mes = “alipayqr://platformapi/startapp?appId=20000120”;饿了么
mes = “alipayqr://platformapi/startapp?appId=20000123”;收钱
mes = “alipayqr://platformapi/startapp?appId=20000125”;首页
mes = “alipayqr://platformapi/startapp?appId=20000126”;免费wifi
mes = “alipayqr://platformapi/startapp?appId=20000130”;滴滴
mes = “alipayqr://platformapi/startapp?appId=20000132”;亲情号
mes = “alipayqr://platformapi/startapp?appId=20000134”;股票自选
mes = “alipayqr://platformapi/startapp?appId=20000135”;火车票
mes = “alipayqr://platformapi/startapp?appId=20000136”;游戏充值
mes = “alipayqr://platformapi/startapp?appId=20000139”;酒店搜索
mes = “alipayqr://platformapi/startapp?appId=20000141”;修改昵称
mes = “alipayqr://platformapi/startapp?appId=20000142”;娱乐宝
mes = “alipayqr://platformapi/startapp?appId=20000143”;火车票汽车票预定
mes = “alipayqr://platformapi/startapp?appId=20000146”;我的淘宝
mes = “alipayqr://platformapi/startapp?appId=20000150”;汇率换算
mes = “alipayqr://platformapi/startapp?appId=20000153”;游戏中心
mes = “alipayqr://platformapi/startapp?appId=20000155”;飞猪
mes = “alipayqr://platformapi/startapp?appId=20000157”;国际机票查询
mes = “alipayqr://platformapi/startapp?appId=20000160”;蚂蚁会员
mes = “alipayqr://platformapi/startapp?appId=20000161”;理财小工具
mes = “alipayqr://platformapi/startapp?appId=20000162”;羊城通
mes = “alipayqr://platformapi/startapp?appId=20000165”;定期理财
mes = “alipayqr://platformapi/startapp?appId=20000161”;指纹手势解锁
mes = “alipayqr://platformapi/startapp?appId=20000168”;年度账单
mes = “alipayqr://platformapi/startapp?appId=20000176”;红包
mes = “alipayqr://platformapi/startapp?appId=20000183”;设置手势密码
mes = “alipayqr://platformapi/startapp?appId=20000161”;指纹手势解锁设定界面
mes = “alipayqr://platformapi/startapp?appId=20000186”;通讯录
mes = “alipayqr://platformapi/startapp?appId=20000161”;绑定智能手环
mes = “alipayqr://platformapi/startapp?appId=20000197”;首页-热门游戏
mes = “alipayqr://platformapi/startapp?appId=20000199”;花呗
mes = “alipayqr://platformapi/startapp?appId=20000205”;亲情圈
mes = “alipayqr://platformapi/startapp?appId=20000218”;黄金
mes = “alipayqr://platformapi/startapp?appId=20000225”;借条
mes = “alipayqr://platformapi/startapp?appId=20000227”;卡包
mes = “alipayqr://platformapi/startapp?appId=20000234”;刷脸
mes = “alipayqr://platformapi/startapp?appId=20000235”;服务提醒
mes = “alipayqr://platformapi/startapp?appId=20000241”;车险服务
mes = “alipayqr://platformapi/startapp?appId=20000243”;总资产
mes = “alipayqr://platformapi/startapp?appId=20000248”;个性签名
mes = “alipayqr://platformapi/startapp?appId=20000252”;朋友模块
mes = “alipayqr://platformapi/startapp?appId=20000255”;账户充值
mes = “alipayqr://platformapi/startapp?appId=20000266”;邮箱账单
mes = “alipayqr://platformapi/startapp?appId=20000288”;聊天室
mes = “alipayqr://platformapi/startapp?appId=20000290”;可能认识的人
mes = “alipayqr://platformapi/startapp?appId=20000298”;证书管理
mes = “alipayqr://platformapi/startapp?appId=20000301”;多设备管理
mes = “alipayqr://platformapi/startapp?appId=20000305”;支付宝内付款码声波付
mes = “alipayqr://platformapi/startapp?appId=20000307”;暗号
```

## 新版沙箱
沙箱在2023年更新过, 老版本已经不再维护, 在老项目复制的代码不能直接用, 需要修改为新版本的密钥

不过使用的方法和参数并没有明显区别

## 神秘代码
```python

import requests
import re

url = 'xxx'
resp = requests.get(url)

ls = re.findall('content="https://render\.alipay\.com/p/yuyan/180020010001207748/h5pay/\*/index\.html\?(.*?)">',
                resp.text)
kvs = {}
if len(ls) > 0:
    query = ls[0]
    sp = query.split('&')
    for row in sp:
        kv = row.split('=')
        kvs[kv[0]] = kv[1]

h5_route_token = kvs['h5_request_token']
query = {
    'appId': '20000125',
    'orderSuffix': f'h5_route_token="{h5_route_token}"&is_h5_route="true"&h5_route_senior="true"'
}

s = f"appId=20000125&orderSuffix={query['orderSuffix']}"
print(s)
```