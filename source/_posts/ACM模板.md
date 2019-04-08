---
title: ACM模板
date: 2019-04-01 23:05:52
categories:
- ACM
tags:
- 模板
---

<!--more-->

## 字符串
### KMP
```cpp
struct KMP{
    int back[maxn];
    void getfail(string line){
        int i,k;
        k = back[0] = -1;
        i = 0;
        while(i < line.size()){
            while(k != -1 && line[i] != line[k]) k = back[k];
            back[++i] = ++k;
        }
    }
    int match(string T,string P){
        int i = 0,k = 0;
        int ret = 0;
        getfail(P);
        while(i < T.size()){
            while(k != -1 && T[i] != P[k]) k = back[k];
            ++i;++k;
            if(k >= P.size()){
                ret++;
                k = back[k];
            }
        }
        return ret;
    }
};
```

### AC自动机
```cpp
/*
insert: 字符串长度
build: 结点数*字符集大小
query: 字符串长度
*/
struct Trie {
    int ch[maxn][26], f[maxn], val[maxn];
    int sz, rt;
    int newnode() {
        memset(ch[sz], -1, sizeof(ch[sz])), val[sz] = 0;
        return sz++;
    }
    void init() { sz = 0, rt = newnode(); }
    int idx(char c) { return c - 'A'; }
    void insert(const char* s) {
        int u = 0;
        for (int i = 0; s[i]; i++) {
            int c = idx(s[i]);
            if (ch[u][c] == -1) ch[u][c] = newnode();
            u = ch[u][c];
        }
        val[u]++;
    }
    void build() {
        queue<int> q;
        f[rt] = rt;
        for (int c = 0; c < 26; c++) {
            if (~ch[rt][c])
                f[ch[rt][c]] = rt, q.push(ch[rt][c]);
            else
                ch[rt][c] = rt;
        }
        while (!q.empty()) {
            int u = q.front();
            q.pop();
            // val[u] |= val[f[u]];
            for (int c = 0; c < 26; c++) {
                if (~ch[u][c])
                    f[ch[u][c]] = ch[f[u]][c], q.push(ch[u][c]);
                else
                    ch[u][c] = ch[f[u]][c];
            }
        }
    }
    int query(const char* s) {
        int u = rt;
        int res = 0;
        for (int i = 0; s[i]; i++) {
            int c = idx(s[i]);
            u = ch[u][c];
            for(int j=u;j!=rt&&~val[j];j=f[j])
                res+=val[j],val[j]=-1;
        }
        return res;
    }
} trie;
```
```cpp
// 指针版
struct Node {
    Node *f, *ch[Types];
    int val;
    Node() {
        val = 0;
        f = nullptr;
        for (int i = 0; i < Types; i++) ch[i] = nullptr;
    }
};
struct Trie {
    vector<Node *> nodes;  // 全部结点，方便删除
    Node *rt;
    void init() {
        rt = new Node();
        nodes.push_back(rt);
    }
    int idx(const char &c) { return c - 'A'; }
    void insert(int id, const char *s) {
        Node *u = rt;
        for (int i = 0, c; s[i]; i++) {
            c = idx(s[i]);
            if (u->ch[c] == nullptr)
                u->ch[c] = new Node(), nodes.push_back(u->ch[c]);
            u = u->ch[c];
        }
        u->val++;
    }
    void build() {
        queue<Node *> q;
        rt->f = rt;
        for (int i = 0; i < Types; i++) {
            if (rt->ch[i] != nullptr)
                rt->ch[i]->f = rt, q.push(rt->ch[i]);
            else
                rt->ch[i] = rt;
        }
        while (!q.empty()) {
            Node *u = q.front();
            q.pop();
            for (int i = 0; i < Types; i++) {
                if (u->ch[i] != nullptr)
                    u->ch[i]->f = u->f->ch[i], q.push(u->ch[i]);
                else
                    u->ch[i] = u->f->ch[i];
            }
        }
    }
    int query(const char *s) {
        int ret = 0;
        Node *u = rt;
        for (int i = 0, c; s[i]; i++) {
            c = idx(s[i]);
            u = u->ch[c];
            for (auto p = u; p != rt && p->val != -1; p = p->f) {
                ret += p->val;
                p->val = -1;
            }
        }
        return ret;
    }
    void delTrie() {
        for (auto &v : nodes) delete v;
        nodes.clear();
    }
} trie;
```

## 数学
### 猜想
哥德巴赫猜想：
- 任何不小于7的奇数，都可以写成三个质数之和(已被证明)
- 任何不小于4的偶数，都可以写成两个质数之和

四色定理：
- 任何一张平面地图只用四种颜色就能使具有公共边界的国家着上不同的颜色

费马大定理：
- 当整数n>2时，关于x,y,z的不定方程$x^n+y^n=z^n$无正整数解

###

### 矩阵快速幂
```cpp
namespace Matrix {
// Matrix mat(row, vec(col));
typedef vector<ll> vec;
typedef vector<vec> mat;
mat mul(mat& A, mat& B) {
    mat C(A.size(), vec(B[0].size()));
    for (int i = 0; i < A.size(); i++)
        for (int k = 0; k < B.size(); k++)
            if (A[i][k])
                for (int j = 0; j < B[0].size(); j++)
                    C[i][j] = (C[i][j] + A[i][k] * B[k][j]) % mod;
    return C;
}
mat Pow(mat A, ll n) {
    mat B(A.size(), vec(A.size()));
    for (int i = 0; i < A.size(); i++) B[i][i] = 1;
    while (n) {
        if (n & 1) B = mul(B, A);
        A = mul(A, A);
        n >>= 1;
    }
    return B;
}
}  // namespace Matrix
```

### 扩展欧几里得
```cpp
// ax + by = gcd(a,b)
// return d = gcd(a,b)
ll ex_gcd(ll a, ll b, ll &x, ll &y) {
    if (a == 0 && b == 0) return -1;  // 无最大公因数
    ll d = a;
    if (b)
        d = ex_gcd(b, a % b, y, x), y -= x * (a / b);
    else
        x = 1, y = 0;
    return d;
}

// X = x + dx * t, Y = y - dy * t
// x是最小非负整数解
bool solve(ll a, ll b, ll c, ll &x, ll &y, ll &dx, ll &dy) {
    ll x0, y0, d;
    d = ex_gcd(a, b, x, y);
    if (d == -1 || c % d) return false;  //无解
    dx = b / d, dy = a / d;
    x = x0 * c / d;
    x = (x % dx + dx) % dx;
    y = (c - a * x) / b;
    return true;
}
```

### 逆元

#### 费马小定理求逆元
```cpp
// 要求: p为质数
ll inv(ll a, ll p) { return Pow(a, p - 2); }
```
#### ex_gcd
```cpp
// 要求: a与p互质
ll inv(ll a, ll p) {
    ll x, y;
    ll d = ex_gcd(a, p, x, y);
    return d == 1 ? (x % p + p) % p : -1;
}
```

#### 阶乘逆元线性递推
```cpp
getInv(int n) {
    f[0] = 1;
    for (int i = 1; i < n; i++) f[i] = f[i - 1] * i % mod;
    inv[n - 1] = Pow(f[n - 1], mod - 2);
    for (int i = n - 2; ~i; i--) inv[i] = inv[i + 1] * (i + 1) % mod;
}
```

#### 逆元线性递推
```cpp
getInv(int n) {
    for (int i = 2; i < n; i++) inv[i] = inv[mod % i] * (mod - mod / i) % mod;
}
```