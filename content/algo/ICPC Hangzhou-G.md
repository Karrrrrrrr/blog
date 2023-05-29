---
title: "2022 ICPC-杭州 G.Subgraph Isomorphism"
date: "2023-05-29T17:26:51+08:00"
---

地址 https://codeforces.com/gym/104090/problem/G

## Subgraph Isomorphism
考点: 树哈希
板子题: https://www.luogu.com.cn/problem/P5043

要点: 

能删边形成同构的条件:

1. 必须是个单环树 (或者本来就是个树, 那就等价于删0条边形成同构了)
2. 在单环的情况下, 把每个环上的点视为根, 对外计算hash值, 如果隔一个的哈希值都相等, 那么就存在同构

## AC代码
```cpp
#include "bits/stdc++.h"

using namespace std;
typedef long long int ll;
typedef array<ll, 2> Hash;

static constexpr Hash p = {223333333, 773333333};
static constexpr Hash mod = {1000000033, 1000002233};

Hash operator*(Hash base, const Hash &p) {
    for (int i = 0; i < base.size(); i++) base[i] *= p[i];
    return base;
}

Hash operator%(Hash base, const Hash &p) {
    for (int i = 0; i < base.size(); i++) base[i] %= p[i];
    return base;
}

Hash operator+(Hash base, const Hash &p) {
    for (int i = 0; i < base.size(); i++) base[i] += p[i];
    return base;
}

Hash operator+(Hash base, const ll &p) {
    for (int i = 0; i < base.size(); i++) base[i] += p;
    return base;
}


void solve() {
    int n, m;
    cin >> n >> m;
    vector<vector<int>> adj(n + 1);
    vector<int> deg(n + 1);
    for (int i = 0; i < m; i++) {
        int u, v;
        cin >> u >> v;
        adj[u].push_back(v);
        adj[v].push_back(u);
        deg[u]++;
        deg[v]++;
    }
    // 该情况下 本身就是个树, 相当于删除0条边形成使得同构, 等价于自己和自己同构
    if (m == n - 1) {
        cout << "YES" << endl;
        return;
    }
    // 该情况下, 存在多个环, 不存在同构
    if (m > n) {
        cout << "NO" << endl;
        return;
    }


    // 拓扑排序 删掉所有的树节点,剩下的必然是个环
    queue<int> q;
    vector<bool> vis(n + 1);
    for (int i = 1; i <= n; i++) {
        if (deg[i] == 1) {
            q.push(i);
        }
    }
    int loopSize = n;
    while (!q.empty()) {
        int cur = q.front();
        q.pop();
        vis[cur] = 1;
        loopSize--;
        for (auto nex: adj[cur]) {
            if (!vis[nex] && --deg[nex] == 1) {
                q.push(nex);
            }
        }
    }


    // 剩下的size就是环的长度

    vector<ll> sz(n + 1);
    // f(x) = (c + sum(g(x))) % m
    function<Hash(int, int)> dfsHash = [&](int cur, int pre) {
        Hash res{};
        sz[cur] = 1;
        vector<Hash> s;
        for (auto nex: adj[cur]) {

            // 递归遍历子树(由于前面拓扑排序删点, 子树必然是合法的树, 不存在环) 获得所有哈希值
            if (vis[nex] && nex != pre) {
                auto childHash = dfsHash(nex, cur);
                s.push_back(childHash);
                sz[cur] += sz[nex];
            }
        }

        // 必须是降序
        sort(s.rbegin(), s.rend());
        for (auto &si: s) {
            for (int i = 0; i < res.size(); i++)
                res = (res * p + si) % mod;
        }
        // 前后顺序不能颠倒 原因未知 
        res = (res * p + sz[cur]) % mod;
        return res;
    };
 

    vector<Hash> h;
    int pre = 0;
    int cur = 0;
    // 任意寻找一个没有遍历的点
    for (int i = 1; i <= n; i++) {
        if (!vis[i]) {
            cur = i;
            break;
        }
    }

    
    // 由于需要按照环的顺序判断hash, 所以必须是按照以下的方法遍历求hash
    for (int i = 0; i < loopSize; i++) {
        auto treeHash = dfsHash(cur, -1);

        h.push_back(treeHash);
        int now = 0;
        // 寻找一个没有遍历过的子节点
        for (auto nex: adj[cur]) {
            if (!vis[nex] && nex != pre) {
                now = nex;
                break;
            }
        }
        pre = cur;
        cur = now;
    }

    // 如果是奇数环, 那么必须每个相同
    // 如果是偶数环, 那么必须隔一个都得相等
    // 等价于 h[i] == h[i+2 % mod] nb!
    for (int i = 0; i < loopSize; i++) {
        if (h[i] != h[(i + 2) % loopSize]) {
            cout << "NO" << endl;
            return;
        }
    }
    cout << "YES" << endl;
}

int main() {
    ios::sync_with_stdio(false), cin.tie(nullptr), cout.tie(nullptr);
    int t = 1;
    cin >> t;
    while (t--) {
        solve();
    }
    return 0;
}


```



## 双模Hash板子
```cpp

typedef long long int ll;
typedef array<ll, 2> Hash;

static constexpr Hash p = {223333333, 773333333};
static constexpr Hash mod = {1000000033, 1000002233};

Hash operator*(Hash base, const Hash &p) {
    for (int i = 0; i < base.size(); i++) base[i] *= p[i];
    return base;
}

Hash operator%(Hash base, const Hash &p) {
    for (int i = 0; i < base.size(); i++) base[i] %= p[i];
    return base;
}

Hash operator+(Hash base, const Hash &p) {
    for (int i = 0; i < base.size(); i++) base[i] += p[i];
    return base;
}

Hash operator+(Hash base, const ll &p) {
    for (int i = 0; i < base.size(); i++) base[i] += p;
    return base;
}

```
