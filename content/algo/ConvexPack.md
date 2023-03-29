---
title: "Andrew"
date: "2023-03-29T17:26:51+08:00"
---

```c++

#include<bits/stdc++.h>

using namespace std;

const int M = 105;

struct Point {
    double x{}, y{};
    Point() {}
    Point(double x, double y) : x(x), y(y) {}
    //	{this->x=x, this->y=y;}
    Point operator+(Point t) const { return {x + t.x, y + t.y}; }
    Point operator-(Point t) const { return {x - t.x, y - t.y}; }
    bool operator==(Point t) const { return x == t.x && y == t.y; }
    bool operator<(const Point &t) const {
        if (x != t.x) return x < t.x;
        return y < t.y;
    }
};

Point a[M], b[M];//a存放所有的点，b存放凸包上的点

//a和b都是向量
double cross(Point a, Point b) {
    /*
        若a * b >0，则a在b的右边；
        若a * b <0,  则a在b的左边；
        若a * b = 0, 则a和b共线(有可能同向，有可能反向)；
    */
    return a.x * b.y - a.y * b.x;
}

double distance(Point a, Point b) {
    return hypot(a.x - b.x, a.y - b.y);
}

int solve(int n) {
    sort(a, a + n);
    n = unique(a, a + n) - a;

    //求下凸包
    int v = 0;
    for (int i = 0; i < n; i++) {
        while (v > 1 && cross(b[v - 1] - b[v - 2], a[i] - b[v - 2]) <= 0.0) v--;
        b[v++] = a[i];
    }

    //求上凸包
    int j = v;
    for (int i = n - 2; i >= 0; i--) {
        while (v > j && cross(b[v - 1] - b[v - 2], a[i] - b[v - 2]) <= 0.0) v--;
        b[v++] = a[i];
    }

    return n > 1 ? v - 1 : v;//v是b里面的顶点数
}

int main() {
    int n;
    while (cin >> n && n) {
        for (int i = 0; i < n; i++)
            cin >> a[i].x >> a[i].y;

        int k = solve(n);
        double ans = 0.0;

        if (k == 1) ans = 0.0;
        else if (k == 2) ans = distance(a[0], a[1]);
        else
            for (int i = 0; i < k; i++)
                ans += distance(b[i], b[(i + 1) % k]);
        // cout<<k<<endl;
        printf("%.2lf\n", ans);
    }

    return 0;
}


```