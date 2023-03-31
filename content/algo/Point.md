---
title: "向量和点重载"
date: "2023-03-29T17:26:51+08:00"
---

```c++

class Point {
public:
    ll x{}, y{};

    // 代替叉乘
    Point operator[](const Point &point) const {
        return {x * point.y - y * point.x};
    }

    ll operator*(const Point &point) const {
        return (x * point.x + point.y * y);
    }

    Point operator-(const Point &point) const {
        return {x - point.x, y - point.y};
    }

    Point operator+(const Point &point) const {
        return {x + point.x, y + point.y};
    }

    Point operator-=(const Point &point) {
        x -= point.x, y -= point.y;
        return *this;
    }

    Point operator+=(const Point &point) {
        x += point.x, y += point.y;
        return *this;
    }

    int operator<(const Point &point) const {
        if (x == point.x) return y < point.y;
        return x < point.x;
    }

    int operator<=(const Point &point) const {
        if (x == point.x) return y <= point.y;
        return x < point.x;
    }

    int operator==(const Point &point) const {
        return *this == point;
    }

    int operator>(const Point &point) const {
        if (x == point.x) return y > point.y;
        return x > point.x;
    }

    int operator>=(const Point &point) const {
        if (x == point.x) return y >= point.y;
        return x > point.x;
    }
};

```