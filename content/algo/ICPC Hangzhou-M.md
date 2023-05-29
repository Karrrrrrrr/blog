---
title: "2022 ICPC-杭州 M. Please Save Pigeland"
date: "2023-05-29T17:26:51+08:00"
---

地址 https://codeforces.com/gym/104090/problem/M

## Please Save Pigeland
考点: 换根, 区间gcd, 区间sum

要点: 
由更相减损法得: gcd(a1,a2,...,an) = gcd(a2-a1, a3-a2, ... , an-an-1)
当a1...ax都要加上某个值的时候, 只需要给 ax+1-ax 修改差值即可(差分思想)

这一部分使用线段树维护

换根:
dfs一次, 记录入栈出栈顺序, 在当前节点入栈出栈之间的, 说明是子节点, 
否则为外部节点, 转移的时候, 子节点的dist都会减少,外部节点的dist都会增加

相当于 1...n都加上dist,  x...y上减少dist*2

然后dfs枚举就行, 期间使用回溯动态修改dist差值


```cpp
#include <bits/stdc++.h>

using namespace std;
typedef long long ll;

const int N = 5e5 + 10;
int n, k;

vector<pair<int, int>> adj[N];

void add(int u, int v, int weight) {
    adj[u].push_back({v, weight});
}

//bool imp[N];
vector<bool> imp(N);
int dfs_in[N], dfs_out[N], node_id[N];
ll dep[N];
int sz[N];

// gcd线段树
template<typename T>
class GCDTree {

    class Node {
    public:
        ll val;
        int l, r;
    };
    
    vector<Node> nodes;
    
    ll _query(int id, int ql, int qr) {
        if (qr < nodes[id].l || ql > nodes[id].r) return 0;
        if (ql <= nodes[id].l && nodes[id].r <= qr) return nodes[id].val;
        return gcd(_query(id << 1, ql, qr), _query(id << 1 | 1, ql, qr));
    }

    void _update(int id, int pos, ll val) {
        if (pos < nodes[id].l || pos > nodes[id].r) return;
        if (nodes[id].l == nodes[id].r) {
            nodes[id].val += val;
            return;
        }
        _update(id << 1, pos, val);
        _update(id << 1 | 1, pos, val);
        pushUp(id);
    }

public:

    void init(int n) {
        nodes.resize(n << 2);
        build(1, 1, n);
    }

    void build(int id, int l, int r) {
        nodes[id].l = l;
        nodes[id].r = r;
        if (l == r) return;

        int mid = (l + r) >> 1;
        build(id << 1, l, mid);
        build(id << 1 | 1, mid + 1, r);
    }

    void pushUp(int id) {
        nodes[id].val = gcd(nodes[id << 1].val,nodes[id << 1 | 1].val);
    }
    
    void update(int pos, ll val) {
        _update(1, pos, val);
    }

    ll query(int ql, int qr) {
        return _query(1, ql, qr);
    }
};

// 线段树
template<typename T = int>
class SegTree {
private:
    class Node {
    public:
        int l = 0;
        int r = 0;
        T val, lazy = 0;
    };

    void _add(int id, int l, int r, T x) {
        if (nodes[id].l > r || nodes[id].r < l) return;

        if (nodes[id].l >= l && nodes[id].r <= r) {
            nodes[id].lazy += x;
            nodes[id].val += (nodes[id].r - nodes[id].l + 1) * x;
            return;
        }

        pushDown(id);
        _add(id << 1, l, r, x);
        _add(id << 1 | 1, l, r, x);
        pushUp(id);
    }

    ll _query(int id, int l, int r) {
        if (nodes[id].l > r || nodes[id].r < l) return 0;
        if (nodes[id].l >= l && nodes[id].r <= r) {
            return nodes[id].val;
        }
        pushDown(id);
        return _query(id << 1, l, r) + _query(id << 1 | 1, l, r);
    }

    void pushUp(int id) {
        nodes[id].val = nodes[id << 1].val + nodes[id << 1 | 1].val;
    }

    void pushDown(int id) {
        if (nodes[id].lazy) {
            nodes[id << 1].lazy += nodes[id].lazy;
            nodes[id << 1].val += nodes[id].lazy * (nodes[id << 1].r - nodes[id << 1].l + 1);
            nodes[id << 1 | 1].lazy += nodes[id].lazy;
            nodes[id << 1 | 1].val += nodes[id].lazy * (nodes[id << 1 | 1].r - nodes[id << 1 | 1].l + 1);
            nodes[id].lazy = 0;
        }

    }

public:
    vector<Node> nodes;

    // 初始化线段树长度, 如果需要初始化值, 那么调用带参build, 不然调用无参build
    SegTree(int n) {
        nodes.resize(n << 2);
    }

    // 允许传入一个vector
    void build(int id, int l, int r, vector<T> &data) {
        build(id, l, r, data.data());
    }

    // 允许传入一个C数组
    void build(int id, int l, int r, T *&data) {
        nodes[id].l = l;
        nodes[id].r = r;

        if (l == r) {
            nodes[id].val = data[l];
            return;
        }
        int mid = (l + r) >> 1;
        build(id << 1, l, mid, data);
        build(id << 1 | 1, mid + 1, r, data);
        pushUp(id);
    }


    void build(int id, int l, int r) {
        nodes[id].l = l;
        nodes[id].r = r;
        nodes[id].val = 0;

        if (l == r) {
            return;
        }
        int mid = (l + r) >> 1;
        build(id << 1, l, mid);
        build(id << 1 | 1, mid + 1, r);
    }

    void add(int left, int right, T x) {
        _add(1, left, right, x);
    }


    T query(int left, int right) {
        return _query(1, left, right);
    }
};

int rev_l[N], rev_r[N];

signed main() {
    ios::sync_with_stdio(false), cin.tie(0), cout.tie(0);
    cin >> n >> k;

    for (int i = 1, x; i <= k; ++i) {
        cin >> x;
        imp[x] = 1;
    }
    for (int i = 1; i < n; ++i) {
        int u, v, w;
        cin >> u >> v >> w;
        add(u, v, w);
        add(v, u, w);
    }

    if (k == 1) {
        cout << 0 << endl;
        return 0;
    }
    vector<int> dfn;
//    for (int i = 1; i <= n; ++i) if (imp[i]) dfn.push_back(dfs_in[i]);
    {
        int time = 0;
        function<void(int, int)> dfs = [&](int u, int fa) {
            ++time;
            dfs_in[u] = time;
            if (imp[u]) dfn.push_back(time);
            node_id[time] = u;
            sz[u] = imp[u]; // 如果是瘟疫区 + 1
            for (auto [v, dist]: adj[u]) {
                // 递归计算子树的距离和尺寸
//                int v = to[i];
                if (v == fa) continue;
                dep[v] = dep[u] + dist;
                dfs(v, u);
                sz[u] += sz[v];
            }
            dfs_out[u] = time;
            // dfs 入栈和出栈中间的 都是子树
        };
        dfs(1, 0);
    }


    sort(dfn.begin(), dfn.end());

    for (int i = 1; i <= n; ++i) {
        rev_l[i] = lower_bound(dfn.begin(), dfn.end(), dfs_in[i]) - dfn.begin() + 1;
        rev_r[i] = upper_bound(dfn.begin(), dfn.end(), dfs_out[i]) - dfn.begin();
    }

    int len = dfn.size();
    SegTree<ll> tree(N);
    GCDTree<ll> gcdTree;
    gcdTree.init(len);
    tree.build(1, 1, N);

    ll ans = 4e18;

    for (int i = 0; i < len; ++i) {
        tree.add(i + 1, i + 1, dep[node_id[dfn[i]]]);
        gcdTree.update(i + 1, dep[node_id[dfn[i]]] - (i == 0 ? 0 : dep[node_id[dfn[i - 1]]]));
    }
    {
        function<void(int, int)> dfs = [&](int u, int fa) {
            ans = min(ans, tree.query(1, len) / gcd(tree.query(1, 1), gcdTree.query(2, len)));
            for (auto [v, dist]: adj[u]) {
                if (v == fa) continue;
                // 如果走到了叶子结点,那么相当于所有人的距离都+dist
                if (sz[v] == 0) {
                    tree.add(1, len, dist);
                    gcdTree.update(1, dist);
                    dfs(v, u);
                    tree.add(1, len, -dist);
                    gcdTree.update(1, -dist);
                } else {
                    // 回溯
                    // 如果是非叶子节点, 那么走到这一步的时候, 对于所有子节点 -dist, 对于所有非子节点+dist
                    tree.add(1, len, dist);
                    tree.add(rev_l[v], rev_r[v], -2 * dist);
                    gcdTree.update(rev_l[v], -2 * dist);
                    gcdTree.update(rev_r[v] + 1, 2 * dist);

                    dfs(v, u);

                    tree.add(1, len, -dist);
                    tree.add(rev_l[v], rev_r[v], 2 * dist);
                    gcdTree.update(rev_l[v], 2 * dist);
                    gcdTree.update(rev_r[v] + 1, -2 * dist);
                }
            }
        };
        dfs(1, 0);
    }
    cout << (ans << 1) << endl;
}


```
