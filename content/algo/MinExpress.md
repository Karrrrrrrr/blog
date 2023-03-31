---
title: "最小表示法"
date: "2023-03-31T17:26:51+08:00"
---

## 最小表示法
```c++
template<typename T=ll>
vector<T> minVector(vector<T> &v) {
    int n = v.size();
    int k = 0, pos = 0, j = 1;
    while (k < n && pos < n && j < n) {
        if (v[(pos + k) % n] == v[(j + k) % n]) {
            k++;
        } else {
            v[(pos + k) % n] > v[(j + k) % n] ? pos = pos + k + 1 : j = j + k + 1;
            if (pos == j) pos++;
            k = 0;
        }
    }
    vector<T> ans(n);
    pos = min(pos, j);
    for (int i = pos; i < pos + n; i++) {
        ans[i - pos] = v[i % n];
    }
    return ans;
}


```