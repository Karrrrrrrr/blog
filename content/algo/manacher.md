---
title: "马拉车算法"
date: "2023-3-20T17:26:51+08:00"
---

## 马拉车算法 ,预处理

```cpp

string manacher_pretreatment(string &s) {
    int n = s.size();
    int m = n * 2 + 3;
    string s2;
    s2.reserve(m);
    s2.push_back('$');
    for (int i = 0; i < n; i++) {
        s2.push_back('#');
        s2.push_back(s[i]);
    }
    s2.push_back('#');
    s2.push_back('!');
    return s2;
}


vector<int> manacher(string &s) {
    //    int[] p = new int[n];
    int id = 0, mx = 0;
    // 最长回文子串的长度
    int maxLength = -1;
    // 最长回文子串的中心位置索引
    int n = s.size();
    vector<int> p(n);
    int index = 0;
    for (int j = 1; j < n - 1; j++) {
        // 参看前文第五部分
        p[j] = mx > j ? min(p[2 * id - j], mx - j) : 1;
        // 向左右两边延伸，扩展右边界
        while (s[j + p[j]] == s[(j - p[j])]) {
            p[j]++;
        }
        // 如果回文子串的右边界超过了mx，则需要更新mx和id的值
        if (mx < p[j] + j) {
            mx = p[j] + j;
            id = j;
        }
        // 如果回文子串的长度大于maxLength，则更新maxLength和index的值
        if (maxLength < p[j] - 1) {
            // 参看前文第三部分
            maxLength = p[j] - 1;
        }
    }
    return p;
}

```