---
title: "并查集"
date: "2023-03-20T17:26:51+08:00"
---

## 并查集
```c++
class UnionFindSet {
    int n;
    vector<int> fa;
public:
    UnionFindSet(int n) {
        this->n = ++n;
        fa.resize(n);
        iota(fa.begin(), fa.end(), 0);
    }
    int find(int x) {
        if (fa[x] == x) return fa[x];
        return fa[x] = find(fa[x]);
    }
    void merge(int x, int y) {
        fa[find(x)] = find(y);
    }
};

```