---
title: "顺丰API"
date: 2024-11-14T10:00:00+08:00

categories: ["后端","快递"]

tags: ["后端","快递"]
---

## 调用顺丰API

官网某些demo写的不详细, 使用requests写一个简单的demo用于记录

```python
import base64
import hashlib
from urllib import parse
import json
import time
import urllib
import uuid
import requests
from urllib.parse import urlencode, quote

env = "development"
checkWord = "fjcg5PGKaNpPSHFAZ4QsCOkV71R3zVci"
code = ''
baseURL = "https://bspgw.sf-express.com/std/service"
if env == "development":
    # 沙箱
    baseURL = "https://sfapi-sbox.sf-express.com/std/service"
    checkWord = "??????"
    # 改为实际的
    code = '??????'

payload = {
    "cargoDetails": [
        {
            "count": 2.365,
            "unit": "个",
            "weight": 6.1,
            "amount": 100.5111,
            "currency": "HKD",
            "name": "护肤品1",
            "sourceArea": "CHN"
        }
    ],
    "contactInfoList": [
        {
            "address": "浙江省宁波市浙江万里学院",
            "contact": "小曾",
            "contactType": 1,
            "country": "CN",
            "postCode": "580058",
            "tel": "4006789888"
        },
        {
            "address": "浙江省宁波市浙江万里学院",
            "company": "顺丰速运",
            "contact": "小邱",
            "contactType": 2,
            "country": "CN",
            "postCode": "580058",
            "tel": "18688806057"
        }
    ],
    "language": "zh_CN",
    "orderId": "OrderNum2020061222321121"
}


def format_sf_data(data):
    # 构造新数据
    new_data = {
        'msgData': data.get('msgData', {}),
        'serviceCode': data.get('serviceCode'),
        'timestamp': data.get('timestamp'),
        'requestID': data.get('requestID'),
        'partnerID': data.get('partnerID'),
        'msgDigest': ''
    }

    raw = json.dumps(new_data['msgData'], ensure_ascii=1) + str(new_data['timestamp']) + checkWord

    # 这里是个坑, 不能直接使用quote对raw编码, 必须是一个字典, 然后截取等号后面的内容
    to_verify_text = urlencode({'': raw})
    to_verify_text = to_verify_text[1:]

    # MD5加密并转为Base64
    md5_hash = hashlib.md5(to_verify_text.encode()).digest()
    # print(md5_hash)
    msgDigest = (base64.b64encode(md5_hash).decode())
    new_data['msgDigest'] = msgDigest
    new_data['msgData'] = json.dumps(new_data['msgData'])

    return new_data


timestamp = str(int(time.time() * 1000))
# timestamp = '1730043755127'
data = format_sf_data({
    'msgData': payload,
    'partnerID': code,
    'requestID': uuid.uuid4().__str__(),
    'serviceCode': 'EXP_RECE_CREATE_ORDER',
    'timestamp': timestamp,
})
# print(data['msgData'])
# print(len(data['msgDigest']))

resp = requests.post(baseURL, headers={
    "Content-type": "application/x-www-form-urlencoded"
}, data=data)
print(data)
print(json.dumps(json.loads(resp.json()['apiResultData']), indent=4, ensure_ascii=0))

```