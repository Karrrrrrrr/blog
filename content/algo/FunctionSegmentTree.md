---
title: "主席树树模板"
date: "2023-03-20T17:26:51+08:00"
---


## 指针式
```c++
Node *build(int l, int r) {
    auto root = newNode();
    root->l = l;
    root->r = r;
    if (l == r) {
        root->sum = 0;
        return root;
    }
    int mid = (l + r) >> 1;
    root->leftChild = build(l, mid);
    root->rightChild = build(mid + 1, r);
    return root;
}


void insert(int l, int r, Node *curNode, Node *pNode, int val) {
    // clone
    *curNode = *pNode;
    if (l == r) {
        curNode->sum++;
        return;
    }
    int mid = l + r >> 1;
    // clone node for reuse

    //
    // 说明更新右, 那么直接复用左边
    if (val > mid) {
        curNode->rightChild = newNode();
        insert(mid + 1, r, curNode->rightChild, pNode->rightChild, val);
    } else {
        curNode->leftChild = newNode();
        insert(l, mid, curNode->leftChild, pNode->leftChild, val);
    }

    curNode->sum = curNode->leftChild->sum +
                   curNode->rightChild->sum;
}

int query(Node *curNode, Node *pNode, int k) {
    if (curNode->l == curNode->r) return curNode->l;
    int sumOfLeft = curNode->leftChild->sum - pNode->leftChild->sum;
    if (k <= sumOfLeft) {
        // find left
        return query(curNode->leftChild, pNode->leftChild, k);
    } else {
        // find right
        return query(curNode->rightChild, pNode->rightChild, k - sumOfLeft);
    }
}

void solve() {
    int n, q;
    cin >> n >> q;
    roots.reserve(n + 10);
    vector<int> v(n);
    map<int, int> mp;
    for (int i = 0; i < n; i++) {
        cin >> v[i];
        mp[v[i]]++;
    }
    int _id = 0;
    vector<int> cache(mp.size() + 1);
    for (auto &item: mp) {
        item.second = ++_id;
        cache[_id] = item.first;
    }
    // 一共有 mp.size()个数字
    int m = (int) mp.size();
    // 初始化一个root
    auto cur = build(1, m);
    roots.push_back(cur);
    auto dbg = [&]() {
        cout << "=====" << endl;
        for (int j = 1; j <= id; j++) {
            cout << "id: " << j << " l: " << nodes[j].l << " r: " << nodes[j].r << " sum: " << nodes[j].sum << endl;
        }
        cout << "=====" << endl;
    };

    for (int i = 0; i < n; i++) {
        auto curNode = newNode();
        auto pNode = roots.back();
        int x = mp[v[i]]; // 离散化之后的值
        roots.push_back(curNode);
        insert(1, m, curNode, pNode, x);
    }

//    dbg();

    while (q--) {
        int l, r, k;
        cin >> l >> r >> k;
        // 两个树相减
        int ans = query(roots[r], roots[l - 1], k);
        cout << cache[ans] << endl;
    }
}

```


## 索引式
```c++
class Node {
public:
    int leftChild{};
    int rightChild{};
    int sum{};
    int l = 0, r = 0;
};

const int N = 2E5;
Node nodes[N * 20];
vector<int> roots;


int id = -1;

int get_id() {
    return ++id;
}

/*
 * 值域线段树
 * 初始化一个线段树, 每次更新添加一条链
 *
 */

int build(int l, int r) {
    auto root = get_id();
    //    nodes[root]
    nodes[root].l = l;
    nodes[root].r = r;
    if (l == r) {
        nodes[root].sum = 0;
        return root;
    }
    int mid = (l + r) >> 1;
    nodes[root].leftChild = build(l, mid);
    nodes[root].rightChild = build(mid + 1, r);
    return root;
}


void insert(int l, int r, int self_id, int p_id, int val) {
    //    cout << "insert x: " << val << endl;
    nodes[self_id] = nodes[p_id];
    if (l == r) {
        nodes[self_id].sum++;
        return;
    }
    int mid = l + r >> 1;
    // clone node for reuse

    //
    // 说明更新右, 那么直接复用左边
    if (val > mid   ) {
        nodes[self_id].rightChild = get_id();
        insert(mid + 1, r, nodes[self_id].rightChild, nodes[p_id].rightChild, val);
    } else {
        nodes[self_id].leftChild = get_id();
        insert(l, mid, nodes[self_id].leftChild, nodes[p_id].leftChild, val);
    }
    nodes[self_id].sum = nodes[nodes[self_id].leftChild].sum + nodes[nodes[self_id].rightChild].sum;
}

int query(int self_id, int p_id, int k) {

    auto &node = nodes[self_id];
    if (node.l == node.r) {
//        cout << "found for x: " << node.sum << endl;
        return node.r;
    }
    int sum_l = nodes[nodes[self_id].leftChild].sum - nodes[nodes[p_id].leftChild].sum;
//    cout << "self_id: " << self_id << " p_id: " << p_id << " k:" << k << " l: " << node.l << " r: " << node.r
//         << " sumL: " << sum_l << endl;

    //    int sum_r = nodes[nodes[self_id].rightChild].sum - nodes[nodes[p_id].rightChild].sum;
    if (k <= sum_l) {
        return query(nodes[self_id].leftChild, nodes[p_id].leftChild, k);
        // find left
    } else {
        // find right
        return query(nodes[self_id].rightChild, nodes[p_id].rightChild, k - sum_l);
    }
}

void solve() {
    roots.reserve(N * 20);
    int n, q;
    cin >> n >> q;
    vector<int> v(n);
    map<int, int> mp;
    for (int i = 0; i < n; i++) {
        cin >> v[i];
        mp[v[i]]++;
    }
    int _id = 0;
    vector<int> cache(mp.size() + 1);
    for (auto &item: mp) {
        item.second = ++_id;
        cache[_id] = item.first;
    }
    //    print(cache);


    //    print(mp);

    // 一共有 mp.size()个数字
    int m = (int) mp.size();
    // 初始化一个root
    auto cur = build(1, m);
    roots.push_back(cur);
    auto dbg = [&]() {
        cout << "=====" << endl;
        for (int j = 1; j <= id; j++) {
            cout << "id: " << j << " l: " << nodes[j].l << " r: " << nodes[j].r << " sum: " << nodes[j].sum << endl;
        }
        cout << "=====" << endl;
    };

    for (int i = 0; i < n; i++) {
        int self_id = get_id();
        int p_id = roots.back();
        int x = mp[v[i]]; // 离散化之后的值
        roots.push_back(self_id);
        //        roots[last] = self_id;
        // cout << "insert x: " << x    << endl;

        //

        insert(1, m, self_id, p_id, x);
    }

    dbg();
//

    //    return;
    //    print(roots);
    while (q--) {
        int l, r, k;
        cin >> l >> r >> k;
        // 两个树相减
        int ans = query(roots[r], roots[l - 1], k);

        cout << cache[ans] << endl;
    }


}

```