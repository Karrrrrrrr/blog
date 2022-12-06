---
title: "拓展欧几里得"
date: 2022-07-06T22:26:42+08:00
---

## 介绍
拓展欧几里得是用来解 ax+by=gcd(a,b) 方程的
返回值是gcd(a,b)

## 费马小定理
 
$$
b ^ { p - 1 } ≡ 1 ( mod p )
$$ 

## 求逆元
 
$$
{ a \over b} ≡ 1 ( mod p)
$$

因为mod是质数

我们根据费马小定理  

$$
{a \over b }  ≡
{a × b ^ { -1 }} × 1 ≡
{a × b ^ { -1 } }  × b ^ { p-1 } ≡
{a × b ^ { p-2 } }
$$

调用快速幂公式即可

```cpp
mod = ? ;
ll power(ll a , ll n){
    ll ans = 1;
    while (n) {
        if( n & 1 ) {
            ans *= a;
            ans %= mod;
        }
        a *= a;
        a %= mod;
        n >>=1;
    }
    return ans;
}


```

## 拓展欧几里得算法实现
### Python
```python
def ex_gcd(dividend, divisor):
    if 0 == divisor:
        return 1, 0, dividend
    x2, y2, remainder = ex_gcd(divisor, dividend % divisor)
    x1 = y2
    y1 = x2 - (dividend // divisor) * y2
    return x1, y1, remainder

```
### C++
```cpp
ll exgcd(ll a, ll b, ll &x, ll &y) {
    if (b == 0) {
        x = 1, y = 0;
        return a;
    }
    ll d = exgcd(b, a % b, y, x);
    y -= a / b * x;
    return d;
}
```