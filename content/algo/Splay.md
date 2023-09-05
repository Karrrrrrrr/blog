---
title: "Splay"
date: "2023-09-06T17:26:51+08:00"
---

# Splay
这是一种平衡二叉树

对于某些题目需要提供以下操作：

1. 插入 x 数；
2. 删除 x 数（若有多个相同的数，只删除一个）；
3. 查询 x 数的排名（若有多个相同的数，输出最小的排名）；
4. 查询排名为 x 的数；
5. 求 x 的前驱（前驱定义为小于 x，且最大的数）；
6. 求 x 的后继（后继定义为大于 x，且最小的数）。



## Code

```c++
#include <cstdio>

const int N = 100005;

struct Splay {
    int root = 0, id = 0, fa[N]{}, ch[N][2]{}, val[N]{}, cnt[N]{}, sz[N]{};

    void pushUp(int x) { sz[x] = sz[ch[x][0]] + sz[ch[x][1]] + cnt[x]; }

    bool get(int x) { return x == ch[fa[x]][1]; }

    void clear(int x) {
        ch[x][0] = ch[x][1] = fa[x] = val[x] = sz[x] = cnt[x] = 0;
    }

    void rotate(int x) {
        int y = fa[x], z = fa[y], chk = get(x);
        ch[y][chk] = ch[x][chk ^ 1];
        if (ch[x][chk ^ 1]) fa[ch[x][chk ^ 1]] = y;
        ch[x][chk ^ 1] = y;
        fa[y] = x;
        fa[x] = z;
        if (z) ch[z][y == ch[z][1]] = x;
        pushUp(y);
        pushUp(x);
    }

    void splay(int x) {
        for (int f = fa[x]; f = fa[x], f; rotate(x))
            if (fa[f]) rotate(get(x) == get(f) ? f : x);
        root = x;
    }

    void insert(int k) {
        if (!root) {
            val[++id] = k;
            cnt[id]++;
            root = id;
            pushUp(root);
            return;
        }
        int cur = root, f = 0;
        while (1) {
            // 等于就结束
            if (val[cur] == k) {
                cnt[cur]++;
                pushUp(cur);
                pushUp(f);
                splay(cur);
                break;
            }
            f = cur;
            cur = ch[cur][val[cur] < k];
            // 如果小就往左,大就往右,
            // 如果当前点不存在 那就插入到这个位置
            if (!cur) {
                val[++id] = k;
                cnt[id]++;
                fa[id] = f;
                ch[f][val[f] < k] = id;
                pushUp(id);
                pushUp(f);
                splay(id);
                break;
            }
        }
    }

    int rank(int k) {
        int ans = 0, cur = root;
        while (1) {
            if (k < val[cur]) {
                cur = ch[cur][0];
            } else {
                ans += sz[ch[cur][0]];
                if (k == val[cur]) {
                    splay(cur);
                    return ans + 1;
                }
                ans += cnt[cur];
                cur = ch[cur][1];
            }
        }
    }

    int kth(int k) {
        int cur = root;
        while (1) {
            if (ch[cur][0] && k <= sz[ch[cur][0]]) {
                cur = ch[cur][0];
            } else {
                k -= cnt[cur] + sz[ch[cur][0]];
                if (k <= 0) {
                    splay(cur);
                    return val[cur];
                }
                cur = ch[cur][1];
            }
        }
    }

    int prev() {
        int cur = ch[root][0];
        if (!cur) return cur;
        while (ch[cur][1]) cur = ch[cur][1];
        splay(cur);
        return cur;
    }

    int next() {
        int cur = ch[root][1];
        if (!cur) return cur;
        while (ch[cur][0]) cur = ch[cur][0];
        splay(cur);
        return cur;
    }

    void del(int k) {
        rank(k);
        if (cnt[root] > 1) {
            cnt[root]--;
            pushUp(root);
            return;
        }
        if (!ch[root][0] && !ch[root][1]) {
            clear(root);
            root = 0;
            return;
        }
        if (!ch[root][0]) {
            int cur = root;
            root = ch[root][1];
            fa[root] = 0;
            clear(cur);
            return;
        }
        if (!ch[root][1]) {
            int cur = root;
            root = ch[root][0];
            fa[root] = 0;
            clear(cur);
            return;
        }
        int cur = root;
        int x = prev();
        fa[ch[cur][1]] = x;
        ch[x][1] = ch[cur][1];
        clear(cur);
        pushUp(root);
    }
} tree;

void work() {
    int n, opt, x;
    for (scanf("%d", &n); n; --n) {
        scanf("%d%d", &opt, &x);
        if (opt == 1)
            tree.insert(x);
        else if (opt == 2)
            tree.del(x);
        else if (opt == 3)
            printf("%d\n", tree.rank(x));
        else if (opt == 4)
            printf("%d\n", tree.kth(x));
        else if (opt == 5)
            tree.insert(x), printf("%d\n", tree.val[tree.prev()]), tree.del(x);
        else
            tree.insert(x), printf("%d\n", tree.val[tree.next()]), tree.del(x);
    }
}


```
 