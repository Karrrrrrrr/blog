---
title: "字符串哈希与矩阵哈希"
date: "2023-04-18T17:26:51+08:00"
---

## 字符串哈希

## 矩阵哈希
我们对于每个矩阵的哈希记作
$$
\sum_{i=1}^N\sum_{j=1}^Ma_{ij} * k_1 ^ { n - i } * k_2 ^ { m-j }
$$

$k_1$, $k_2$ 分别是行列进制值, 一般取2333, 23333, $a_{ij}$是原矩阵的值

注意点: 涉及到前缀和的计算, 索引一般从1开始,而不是0

$k_1$, $k_2$ 的幂计算需要提前预处理 

那么, 对于一个矩阵右下角为 (x,y), 宽高为(w,h)的矩阵哈希值为
$$
{H}(x,y) + {H}(x-{h} , y-{w}) * {k_1}^{h} * {k_2}^{w} - H(x-{h},y)*k_1^h - H(x,y-{w})*k_{2}^w
$$

题意：给一个矩阵，进行一次询问，询问给定一个矩阵，问询问的矩阵在原矩阵里匹配上了几次。



```c++

typedef unsigned long long int Value;
typedef vector<Value> Row;
typedef vector<Row> Matrix;

const int bx = 233, by = 2333;
const int MAX_P = 1e4;
ull px[MAX_P] = {1};
ull py[MAX_P] = {1};

static int _ = []() {
    for (int i = 1; i < MAX_P; i++) px[i] = px[i - 1] * bx, py[i] = py[i - 1] * by;
    return 0;
}();

template<typename T=Value>
class Mat {
    Matrix _mat;

    Matrix newMatrix(int n, int m, Value val = 0) {
        return Matrix(n + 1, Row(m + 1, val));
    }

public:
    Mat(int n, int m, T val = 0) {
        _mat = newMatrix(n, m, val);
    }

    Mat(Matrix &mat) {
        _mat = mat;
    }
    
    // 返回一个哈希之后的矩阵
    Mat calcHash() {
        auto hashMatrix = _mat;
        int n = _mat.size() - 1;
        int m = _mat[0].size() - 1;
        for (int i = 1; i <= n; i++) {
            for (int j = 1; j <= m; j++) {
                hashMatrix[i][j] = (hashMatrix[i - 1][j] * bx
                                    + hashMatrix[i][j - 1] * by
                                    - hashMatrix[i - 1][j - 1] * bx * by + _mat[i][j]);
            }
        }

        return Mat(hashMatrix);
    }

    // 计算右下角坐标为 (x,y) 高宽为 (height,width)的矩阵哈希值
    ull subHash(int x, int y, int height, int width) {
        if (x - height < 0 || y - width < 0) return -1;
        ull hash =
                _mat[x][y]
                + _mat[x - height][y - width] * px[height] * py[width]
                - _mat[x][y - width] * py[width]
                - _mat[x - height][y] * px[height];
        return hash;
    }


    Row &operator[](int idx) { return _mat[idx]; }

    size_t size() { return _mat.size(); }

    Row &back() { return _mat.back(); }
};


```


###  模板题以及AC代码
地址 https://www.acwing.com/problem/content/description/158/


```c++
#include <bits/stdc++.h>

using namespace std;
#define endl '\n'
typedef long long int ll;
typedef unsigned long long int ull;
const ll inf = 0x7fffffff;
const ll mod = 998244353;

void init() {
#ifdef endl
    ios::sync_with_stdio(false), cin.tie(0), cout.tie(0);
#endif
}

typedef unsigned long long int Value;
typedef vector<Value> Row;
typedef vector<Row> Matrix;

const int bx = 233, by = 2333;
const int MAX_P = 1e4;
ull px[MAX_P] = {1};
ull py[MAX_P] = {1};

static int _ = []() {
    for (int i = 1; i < MAX_P; i++) px[i] = px[i - 1] * bx, py[i] = py[i - 1] * by;
    return 0;
}();

template<typename T=Value>
class Mat {
    Matrix _mat;

    Matrix newMatrix(int n, int m, Value val = 0) {
        return Matrix(n + 1, Row(m + 1, val));
    }

public:
    Mat(int n, int m, T val = 0) {
        _mat = newMatrix(n, m, val);
    }

    Mat(Matrix &mat) {
        _mat = mat;
    }

    Mat calcHash() {
        auto hashMatrix = _mat;
        int n = _mat.size() - 1;
        int m = _mat[0].size() - 1;
        for (int i = 1; i <= n; i++) {
            for (int j = 1; j <= m; j++) {
                hashMatrix[i][j] = (hashMatrix[i - 1][j] * bx
                                    + hashMatrix[i][j - 1] * by
                                    - hashMatrix[i - 1][j - 1] * bx * by + _mat[i][j]);
            }
        }

        return Mat(hashMatrix);
    }

    ull subHash(int x, int y, int height, int width) {
        if (x - height < 0 || y - width < 0) return -1;
        ull hash =
                _mat[x][y]
                + _mat[x - height][y - width] * px[height] * py[width]
                - _mat[x][y - width] * py[width]
                - _mat[x - height][y] * px[height];
        return hash;
    }


    Row &operator[](int idx) { return _mat[idx]; }

    size_t size() { return _mat.size(); }

    Row &back() { return _mat.back(); }
};







//Matrix calcHash(Matrix &mat) {
//    auto hashMatrix = mat;
//    int n = mat.size() - 1;
//    int m = mat[0].size() - 1;
//    for (int i = 1; i <= n; i++) {
//        for (int j = 1; j <= m; j++) {
//            hashMatrix[i][j] = (hashMatrix[i - 1][j] * bx + hashMatrix[i][j - 1] * by
//                                - hashMatrix[i - 1][j - 1] * bx * by + mat[i][j]);
//        }
//    }
//    return hashMatrix;
//}

void solve() {
    int n, m, N, M;
    cin >> N >> M >> n >> m;
    auto matrix = Mat(N, M, 0);

    for (int i = 1; i <= N; i++) {
        for (int j = 1; j <= M; j++) {
            char c;
            cin >> c;
            //            c -= '0';
            matrix[i][j] = c;
        }
    }
    auto hashMatrix = matrix.calcHash();

    map<ull, ull> hashMap;
    for (int i = 1; i <= N; i++) {
        for (int j = 1; j <= M; j++) {
            if (i - n < 0 || j - m < 0) continue;
            ull hash = hashMatrix.subHash(i, j, n, m);
            hashMap[hash] = 1;
        }
    }
    int t;
    cin >> t;
    while (t--) {
        auto subMat = Mat(n, m, 0);
        for (int i = 1; i <= n; i++) {
            for (int j = 1; j <= m; j++) {
                char c;
                cin >> c;
                subMat[i][j] = c;
            }
        }
        auto hashSubMat = subMat.calcHash();
        cout << hashMap[hashSubMat.back().back()] << endl;
    }
}

int main() {
    init();
    int T = 1;
    while (T--) {
        solve();
    }
    return 0;
}

/*
3 3 1 3
1 1 1
0 1 0
1 0 1

3
1 1 1
1 0 1
1 1 0
 *
 */
```