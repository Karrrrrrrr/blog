---
title: "二维树状数组-BinaryIndexedTree"
date: "2023-03-29T17:26:51+08:00"
---

```c++
class BinaryIndexedTree {
    static const int M = 105;
    ll t1[M][M]{}, t2[M][M]{}, t3[M][M]{}, t4[M][M]{};

    int lowbit(int x) {
        return x & -x;
    }
    
    void _add(int x, int y, ll val) {
        const int n = 101, m = 101;
        for (int X = x; X <= n; X += lowbit(X))
            for (int Y = y; Y <= m; Y += lowbit(Y)) {
                t1[X][Y] += val;
                t2[X][Y] += val * x;  // 注意是 val * x 而不是 val * X，后面同理
                t3[X][Y] += val * y;
                t4[X][Y] += val * x * y;
            }
    }

public:
    void addRange(int x1, int y1, int x2, int y2, ll val) {
        _add(x1, y1, val);
        _add(x1, y2 + 1, -val);
        _add(x2 + 1, y1, -val);
        _add(x2 + 1, y2 + 1, val);
    }

    void add(int x1, int y1, ll val) {
        addRange(x1, y1, x1, y1, val);
    }

    ll query(int x, int y) {
        ll res = 0;
        for (int i = x; i; i -= lowbit(i))
            for (int j = y; j; j -= lowbit(j))
                res += (x + 1) * (y + 1) * t1[i][j] - (y + 1) * t2[i][j] -
                       (x + 1) * t3[i][j] + t4[i][j];
        return res;
    }

    ll queryRange(int xa, int ya, int xb, int yb) {
        return query(xb, yb) -
               query(xb, ya - 1) -
               query(xa - 1, yb) +
               query(xa - 1, ya - 1);
    }
};

```