---
title: "Karrrrrr's 线段树"
date: "2023-03-26T17:26:51+08:00"
---

## 最顺手的线段树
```
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

    void _add(int id, int l, int r, T x = 1) {
        if (nodes[id].l > r || nodes[id].r < l) return;

        if (nodes[id].l >= l && nodes[id].r <= r) {
            nodes[id].lazy += x;
            nodes[id].val += (nodes[id].r - nodes[id].l + 1) * x;
            return;
        }

        push_down(id);
        add(id << 1, l, r);
        add(id << 1 | 1, l, r);
        push_up(id);
    }

    ll _query(int id, int l, int r) {
        if (nodes[id].l > r || nodes[id].r < l) return 0;
        if (nodes[id].l >= l && nodes[id].r <= r) {
            return nodes[id].val;
        }
        push_down(id);
        return _query(id << 1, l, r) + _query(id << 1 | 1, l, r);
    }

public:
    vector<Node> nodes;

    SegTree(int n) {
        nodes.resize(n << 2);
        build(1, 1, n);
    }

    void build(int id, int l, int r, vector<int> &data) {
        nodes[id].l = l;
        nodes[id].r = r;

        if (l == r) {
            nodes[id].val = data[l];
            return;
        }
        int mid = (l + r) >> 1;
        build(id << 1, l, mid);
        build(id << 1 | 1, mid + 1, r);
        push_up(id);
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

    void push_up(int id) {
        nodes[id].val = nodes[id << 1].val + nodes[id << 1 | 1].val;
    }

    void push_down(int id) {
        if (nodes[id].lazy) {
            nodes[id << 1].lazy += nodes[id].lazy;
            nodes[id << 1].val += nodes[id].lazy * (nodes[id << 1].r - nodes[id << 1].l + 1);
            nodes[id << 1 | 1].lazy += nodes[id].lazy;
            nodes[id << 1 | 1].val += nodes[id].lazy * (nodes[id << 1 | 1].r - nodes[id << 1 | 1].l + 1);
            nodes[id].lazy = 0;
        }

    }

    T query(int left, int right) {
        return _query(1, left, right);
    }
};


```