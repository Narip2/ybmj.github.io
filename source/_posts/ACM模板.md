---
title: ACM模板
date: 2019-04-01 23:05:52
categories:
  - ACM
tags:
  - 模板
---

<!--more-->
## 头文件
```cpp
// ybmj
#include <bits/stdc++.h>
using namespace std;
#define lson (rt << 1)
#define rson (rt << 1 | 1)
#define lson_len (len - (len >> 1))
#define rson_len (len >> 1)
#define pb(x) push_back(x)
#define clr(a, x) memset(a, x, sizeof(a))
#define mp(x, y) make_pair(x, y)
using ll = long long;
using pii = pair<int,int>;
#define my_unique(a) a.resize(distance(a.begin(), unique(a.begin(), a.end())))
#define my_sort_unique(c) (sort(c.begin(), c.end())), my_unique(c)
const int INF = 0x3f3f3f3f;
const int NINF = 0xc0c0c0c0;
const double PI = acos(-1.0);

int main() {
//	/*
#ifndef ONLINE_JUDGE
    freopen("1.in", "r", stdin);
    freopen("1.out", "w", stdout);
#endif
//	*/
    std::ios::sync_with_stdio(false);

}
```
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

### AC 自动机

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

### 后缀数组

```cpp
/*
sa[i]：表示排名为 i 的后缀的起始位置
rank[i]：表示从 i 开始的后缀的排名
height[i] = LCP(suffix(sa[i-1]), suffix(sa[i])) ：排名为 i-1 和排名为 i 的两个后缀的最长公共前缀
h[i] = height[rank[i]] ：起始位置为 i 的后缀和它前一名的后缀的最长公共前缀
build 复杂度:O(nlogn)
// 需要保证s[n]为之前没有出现过的最小的字符。必要时手动添加，或者自动'\0'。
// sa下标从1开始编号，sa[0]表示加入的最小字符'\0'
*/
struct SA {
    char s[maxn];
    int sa[maxn], t[maxn], t2[maxn], c[maxn], rank[maxn], height[maxn];
    int dp[maxn][30];
    void build(int m, int n) {  // [0,m-1]字符集, [0,n-1] 字符串
        n++;
        int *x = t, *y = t2;
        for (int i = 0; i < m; i++) c[i] = 0;
        for (int i = 0; i < n; i++) c[x[i] = s[i]]++;
        for (int i = 1; i < m; i++) c[i] += c[i - 1];
        for (int i = n - 1; ~i; i--) sa[--c[x[i]]] = i;
        for (int k = 1; k <= n; k <<= 1) {
            int p = 0;
            for (int i = n - k; i < n; i++) y[p++] = i;
            for (int i = 0; i < n; i++)
                if (sa[i] >= k) y[p++] = sa[i] - k;
            for (int i = 0; i < m; i++) c[i] = 0;
            for (int i = 0; i < n; i++) c[x[y[i]]]++;
            for (int i = 1; i < m; i++) c[i] += c[i - 1];
            for (int i = n - 1; ~i; i--) sa[--c[x[y[i]]]] = y[i];
            swap(x, y);
            p = 1;
            x[sa[0]] = 0;
            for (int i = 1; i < n; i++)
                x[sa[i]] =
                    y[sa[i - 1]] == y[sa[i]] && y[sa[i - 1] + k] == y[sa[i] + k]
                        ? p - 1
                        : p++;
            if (p >= n) break;
            m = p;
        }
        n--;
        int k = 0;
        for (int i = 0; i <= n; i++) rank[sa[i]] = i;
        for (int i = 0; i < n; i++) {
            if (k) k--;
            int j = sa[rank[i] - 1];
            while (s[i + k] == s[j + k]) k++;
            height[rank[i]] = k;
        }
    }
    void initRmq(int n) {
        for (int i = 1; i <= n; i++) dp[i][0] = height[i];
        for (int j = 1; (1 << j) <= n; j++)
            for (int i = 1; i + (1 << j) - 1 <= n; i++)
                dp[i][j] = min(dp[i][j - 1], dp[i + (1 << (j - 1))][j - 1]);
    }
    int rmq(int l, int r) {
        int k = 31 - __builtin_clz(r - l + 1);
        return min(dp[l][k], dp[r - (1 << k) + 1][k]);
    }
    int lcp(int a, int b) {
        // 返回起始地址为a和b的两个后缀串的lcp
        a = rank[a], b = rank[b];
        if (a > b) swap(a, b);
        return rmq(a + 1, b);
    }
} sa;
```

### Manacher 最长回文子串

```cpp
const int maxn = 1e6+5;
char Ma[maxn*2];    //插入新字符后的字符串
int Mp[maxn*2];     //以当前位置为对称轴的回文半径

void Manacher(string &s){
    int l = 0;
    Ma[l++] = '$';      //防止越界
    Ma[l++] = '#';
    for(int i=0;i<s.size();i++){
        Ma[l++] = s[i];
        Ma[l++] = '#';
    }
    Ma[l] = 0;      //结尾设置为空字符，防止越界
    int mx = 1,id = 1;
    for(int i=1;i<l;i++){
        Mp[i] = mx > i ? min(Mp[2*id-i],mx-i) : 1;
        while(Ma[i+Mp[i]] == Ma[i-Mp[i]]) Mp[i] ++; //扩展
        if(i + Mp[i] > mx){
            mx = i + Mp[i];
            id = i;
        }
    }
}
```

## 数学

### 小知识

**哥德巴赫猜想：**

- 任何不小于 7 的奇数，都可以写成三个质数之和(已被证明)
- 任何不小于 4 的偶数，都可以写成两个质数之和

**四色定理：**

- 任何一张平面地图只用四种颜色就能使具有公共边界的国家着上不同的颜色

**费马大定理：**

- 当整数 n>2 时，关于 x,y,z 的不定方程$x^n+y^n=z^n$无正整数解

**费马小定理：**

- 对于质数 p，对于任意整数 a，均满足$a^p \equiv a$ (mod p)

**欧拉函数**

- $\phi(m) = \{a: 1 \leq a \leq m, gcd(a,m) = 1 \}$

**欧拉定理：**

- 若正整数 a，n 互质，则$a ^ {\phi(n)} \equiv 1 \quad mod(n)$

**欧拉定理推论（降幂）：**

- 若正整数 a，n 互质，那么对于任意正整数 b，有$a^b \equiv a ^{b \% \phi(n)}$ mod(n)

**威尔逊定理：**

- p 为素数的充要条件为$(p-1)! \equiv -1 \quad mod(p)$

**二次探测定理：**

- 如果 p 是一个素数，且$0 < x < p$，则方程$x^2 \equiv 1 \quad mod(p)$的解为 1 和 p-1

**海伦公式**

- 三角形的面积公式$S = \sqrt{p(p-a)(p-b)(p-c)}$，其中$p = \frac{a+b+c}{2}$

**婆罗摩笈多公式**

- 圆内接四边形面积公式$S = \sqrt{(p-a)(p-b)(p-c)(p-d)}$，其中$p = \frac{a+b+c+d}{2}$
- 一般四边形面积公式$S = \sqrt{(p-a)(p-b)(p-c)(p-d) - abcd\cos^2 \theta }$，其中$\theta$为四边形一对角和的一半。
- 对于给定的四条边长，所有组成的四边形中圆内接四边形的面积最大

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

### 类欧几里得
$f = \sum^n_{i=0} \lfloor \frac{ai+b}{c} \rfloor $
$g = \sum^n_{i=0} \lfloor \frac{ai+b}{c} \rfloor ^2$
$h = \sum^n_{i=0} i\lfloor \frac{ai+b}{c} \rfloor $

```cpp
namespace likeGcd {
using ll = long long;
const ll mod = 998244353;
const ll inv2 = 499122177;
const ll inv6 = 166374059;
ll n, a, b, c;

struct node {
  ll f, g, h;
};

inline ll S1(ll n) { return n * (n + 1) % mod * inv2 % mod; }

inline ll S2(ll n) {
  return n * (n + 1) % mod * (2 * n + 1) % mod * inv6 % mod;
}

inline node solve(ll a, ll b, ll c, ll n) {
  ll t1 = a / c, t2 = b / c, s1 = S1(n), s2 = S2(n), m = (a * n + b) / c;
  node ans, now;
  ans.f = ans.g = ans.h = 0;
  if (!n) {
    ans.f = t2;
    ans.g = t2 * t2 % mod;
    return ans;
  }
  if (!a) {
    ans.f = (n + 1) * t2 % mod;
    ans.g = (n + 1) * t2 % mod * t2 % mod;
    ans.h = t2 * s1 % mod;
    return ans;
  }
  if (a >= c || b >= c) {
    now = solve(a % c, b % c, c, n);
    ans.f = (now.f + t1 * s1 + (n + 1) * t2) % mod;
    ans.g = (now.g + 2 * t1 * now.h + 2 * t2 * now.f + t1 * t1 % mod * s2 +
             2 * t1 * t2 % mod * s1 + (n + 1) * t2 % mod * t2) %
            mod;
    ans.h = (now.h + t1 * s2 + t2 * s1) % mod;
    return ans;
  }
  now = solve(c, c - b - 1, a, m - 1);
  ans.f = (m * n - now.f) % mod;
  ans.f = (ans.f + mod) % mod;
  ans.g = (m * m % mod * n - 2 * now.h - now.f);
  ans.g = (ans.g + mod) % mod;
  ans.h = (m * s1 - now.g * inv2 - now.f * inv2) % mod;
  ans.h = (ans.h + mod) % mod;
  return ans;
}

};  // namespace likeGcd
```
### CRT

```cpp
// mod两两互质
// 通解 re + k * M
// 返回最小非负整数解
void crt(ll r[], ll m[], ll n, ll &re, ll &M) {
    M = 1, re = 0;
    for (int i = 0; i < n; i++) M *= m[i];
    for (int i = 0; i < n; i++) {
        ll x, y,  tm = M / m[i];
        ll d = exgcd(tm, m[i], x, y);
        re = (re + tm * x * r[i]) % M;
    }
    re = (re + M) % M;
}
```

#### EXCRT

```cpp
// mod不满足两两互质
// 通解为 re + k*M
// 返回最小非负整数解
bool excrt(ll r[], ll m[], ll n, ll &re, ll &M) {
    ll x, y;
    M = m[0], re = r[0];
    for (int i = 1; i < n; i++) {
        ll d = exgcd(M, m[i],  x, y);
        if ((r[i] - re) % d != 0) return 0;
        x = (r[i] - re) / d * x % (m[i] / d);
        re += x * M;
        M = M / d * m[i];
        re %= M;
    }
    re = (re + M) % M;
    return 1;
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

### 快速乘

```
inline ll Mul(ll a, ll b, ll m) {
    if (m <= (ll)1e9)
        return a * b % m;
    else if (m <= (ll)1e12)
        return (((a * (b >> 20) % m) << 20) + (a * (b & ((1 << 20) - 1)))) % m;
    else {
        ll d = (ll)floor(a * (long double)b / m + 0.5);
        ll ret = (a * b - d * m) % m;
        if (ret < 0) ret += m;
        return ret;
    }
}
```

### Miller Rabin 大素数判定

```cpp
// O(slogn)内判定素数
// s为判定次数，错误率为(1/4)^s
bool Miller_Rabin(ull n, int s) {
    if (n == 2) return 1;
    if (n < 2 || !(n & 1)) return 0;
    int t = 0;
    ull x, y, u = n - 1;
    while ((u & 1) == 0) t++, u >>= 1;
    for (int i = 0; i < s; i++) {
        ull a = rand() % (n - 1) + 1;
        ull x = Pow(a, u, n);
        for (int j = 0; j < t; j++) {
            // ull y = Mul(x, x, n);
            ull y = __int128(x) * __int128(x) % n;
            if (y == 1 && x != 1 && x != n - 1) return 0;
            x = y;
        }
        if (x != 1) return 0;
    }
    return 1;
}
```

### FFT

```cpp
namespace fft {
typedef double db;

struct cp {
    db x, y;
    cp() { x = y = 0; }
    cp(db x, db y) : x(x), y(y) {}
};

inline cp operator+(cp a, cp b) { return cp(a.x + b.x, a.y + b.y); }
inline cp operator-(cp a, cp b) { return cp(a.x - b.x, a.y - b.y); }
inline cp operator*(cp a, cp b) {
    return cp(a.x * b.x - a.y * b.y, a.x * b.y + a.y * b.x);
}
inline cp conj(cp a) { return cp(a.x, -a.y); }

int base = 1;
vector<cp> roots = {{0, 0}, {1, 0}};
vector<int> rev = {0, 1};

const db PI = acosl(-1.0);

void ensure_base(int nbase) {
    if (nbase <= base) return;
    rev.resize(static_cast<unsigned long>(1 << nbase));
    for (int i = 0; i < (1 << nbase); i++)
        rev[i] = (rev[i >> 1] >> 1) + ((i & 1) << (nbase - 1));
    roots.resize(static_cast<unsigned long>(1 << nbase));
    while (base < nbase) {
        db angle = 2 * PI / (1 << (base + 1));
        for (int i = 1 << (base - 1); i < (1 << base); i++) {
            roots[i << 1] = roots[i];
            db angle_i = angle * (2 * i + 1 - (1 << base));
            roots[(i << 1) + 1] = cp(cos(angle_i), sin(angle_i));
        }
        base++;
    }
}

void fft(vector<cp>& a, int n = -1) {
    if (n == -1) n = a.size();
    assert((n & (n - 1)) == 0);
    int zeros = __builtin_ctz(n);
    ensure_base(zeros);
    int shift = base - zeros;
    for (int i = 0; i < n; i++)
        if (i < (rev[i] >> shift)) swap(a[i], a[rev[i] >> shift]);
    for (int k = 1; k < n; k <<= 1)
        for (int i = 0; i < n; i += 2 * k)
            for (int j = 0; j < k; j++) {
                cp z = a[i + j + k] * roots[j + k];
                a[i + j + k] = a[i + j] - z;
                a[i + j] = a[i + j] + z;
            }
}

vector<cp> fa, fb;

template <class T>
vector<T> multiply(vector<T>& a, vector<T>& b) {
    int need = a.size() + b.size() - 1;
    int nbase = 0;
    while ((1 << nbase) < need) nbase++;
    ensure_base(nbase);
    int sz = 1 << nbase;
    if (sz > (int)fa.size()) fa.resize(static_cast<unsigned long>(sz));
    for (int i = 0; i < sz; i++) {
        T x = (i < (int)a.size() ? a[i] : 0);
        T y = (i < (int)b.size() ? b[i] : 0);
        fa[i] = cp(x, y);
    }
    fft(fa, sz);
    cp r(0, -0.25 / sz);
    for (int i = 0; i <= (sz >> 1); i++) {
        int j = (sz - i) & (sz - 1);
        cp z = (fa[j] * fa[j] - conj(fa[i] * fa[i])) * r;
        if (i != j) {
            fa[j] = (fa[i] * fa[i] - conj(fa[j] * fa[j])) * r;
        }
        fa[i] = z;
    }
    fft(fa, sz);
    vector<T> res(static_cast<unsigned long>(need));
    for (int i = 0; i < need; i++) res[i] = fa[i].x + 0.5;
    return res;
}
};  // namespace fft

namespace ntt {
int Pow(int a, int b, int p) {
  int ret = 1;
  while (b) {
    if (b & 1) ret = 1LL * ret * a % p;
    a = 1LL * a * a % p;
    b >>= 1;
  }
  return ret;
}
const int mod = 119 << 23 | 1;
const int G = 3;
int wn[20];
void getwn() {  //  千万不要忘记
  for (int i = 0; i < 20; i++) wn[i] = Pow(G, (mod - 1) / (1 << i), mod);
}
void change(int y[], int len) {
  for (int i = 1, j = len / 2; i < len - 1; i++) {
    if (i < j) swap(y[i], y[j]);
    int k = len / 2;
    while (j >= k) j -= k, k /= 2;
    if (j < k) j += k;
  }
}
void ntt(int y[], int len, int on) {
  change(y, len);
  for (int h = 2, id = 1; h <= len; h <<= 1, id++) {
    for (int j = 0; j < len; j += h) {
      int w = 1;
      for (int k = j; k < j + h / 2; k++) {
        int u = y[k] % mod;
        int t = 1LL * w * (y[k + h / 2] % mod) % mod;
        y[k] = (u + t) % mod, y[k + h / 2] = ((u - t) % mod + mod) % mod;
        w = 1LL * w * wn[id] % mod;
      }
    }
  }
  if (on == -1) {
    //  原本的除法要用逆元
    int inv = Pow(len, mod - 2, mod);
    for (int i = 1; i < len / 2; i++) swap(y[i], y[len - i]);
    for (int i = 0; i < len; i++) y[i] = 1LL * y[i] * inv % mod;
  }
}
/*
void multiply(int x[], int y[], int n, int m) {
  // n,m 表示长度 （阶数要加一）
  int len = m + n;
  while (len <= (m + n)) len <<= 1;
  for (int i = 0; i < len; i++) A[i] = B[i] = 0;
  for (int i = 0; i <= n; i++) A[i] = x[i];
  for (int i = 0; i <= m; i++) B[i] = y[i];
  ntt(A, len, 1);
  ntt(B, len, 1);
  for (int i = 0; i < len; i++) A[i] = 1LL * A[i] * B[i] % mod;
  ntt(A, len, -1);
}
*/
}  // namespace ntt

namespace fft {
const double PI = acos(-1.0);
//复数结构体
struct Complex {
  double x, y;  //实部和虚部 x+yi
  Complex(double _x = 0.0, double _y = 0.0) { x = _x, y = _y; }
  Complex operator-(const Complex& b) const {
    return Complex(x - b.x, y - b.y);
  }
  Complex operator+(const Complex& b) const {
    return Complex(x + b.x, y + b.y);
  }
  Complex operator*(const Complex& b) const {
    return Complex(x * b.x - y * b.y, x * b.y + y * b.x);
  }
};
void change(Complex y[], int len) {
  for (int i = 1, j = len / 2; i < len - 1; i++) {
    if (i < j) swap(y[i], y[j]);
    int k = len / 2;
    while (j >= k) j -= k, k /= 2;
    if (j < k) j += k;
  }
}
/*
 * len必须为2^k形式，
 * on==1时是DFT，on==-1时是IDFT
 */
void fft(Complex y[], int len, int on) {
  change(y, len);
  for (int h = 2; h <= len; h <<= 1) {
    Complex wn(cos(-on * 2 * PI / h), sin(-on * 2 * PI / h));
    for (int j = 0; j < len; j += h) {
      Complex w(1, 0);
      for (int k = j; k < j + h / 2; k++) {
        Complex u = y[k];
        Complex t = w * y[k + h / 2];
        y[k] = u + t, y[k + h / 2] = u - t;
        w = w * wn;
      }
    }
  }
  if (on == -1)
    for (int i = 0; i < len; i++) y[i].x /= len;
}
/*
void multiply(int x[], int y[], int n, int m) {
  // n,m 表示长度 （阶数要加一）
  int len = m + n;
  while (len <= (m + n)) len <<= 1;
  for (int i = 0; i < len; i++) A[i] = B[i] = Complex(0, 0);
  for (int i = 0; i <= n; i++) A[i] = Complex(x[i], 0);
  for (int i = 0; i <= m; i++) B[i] = Complex(y[i], 0);
  fft(A, len, 1);
  fft(B, len, 1);
  for (int i = 0; i < len; i++) A[i] = 1LL * A[i] * B[i] % mod;
  fft(A, len, -1);
  for (int i = 0; i < len; i++) C[i] = int(A[i].x + 0.5);
  while (C[len] == 0 && len > 0) len--;
}
*/
}  // namespace fft

namespace fwt {
void fwt(int f[], int m) {
  int n = __builtin_ctz(m);
  for (int i = 0; i < n; ++i)
    for (int j = 0; j < m; ++j)
      if (j & (1 << i)) {
        int l = f[j ^ (1 << i)], r = f[j];
        f[j ^ (1 << i)] = l + r, f[j] = l - r;
        // or: f[j] += f[j ^ (1 << i)];
        // and: f[j ^ (1 << i)] += f[j];
      }
}
void ifwt(int f[], int m) {
  int n = __builtin_ctz(m);
  for (int i = 0; i < n; ++i)
    for (int j = 0; j < m; ++j)
      if (j & (1 << i)) {
        int l = f[j ^ (1 << i)], r = f[j];
        f[j ^ (1 << i)] = (l + r) / 2, f[j] = (l - r) / 2;
        // 如果有取模需要使用逆元
        // or: f[j] -= f[j ^ (1 << i)];
        // and: f[j ^ (1 << i)] -= f[j];
      }
}
}  // namespace fwt
```



### 线性筛因数个数
```cpp
const int maxn = 1e7 + 5;
bool vis[maxn];
int prime[maxn], d[maxn], num[maxn], tot;
// d[i]: i的约数个数， num[i]: i的最小质因数的个数
void init() {
  memset(vis, 0, sizeof(vis));
  d[1] = 1;
  for (ll i = 2; i < maxn; i++) {
    if (!vis[i]) {
      prime[tot++] = i;
      d[i] = 2;
      num[i] = 1;
    }
    for (ll k = 0; k < tot && i * prime[k] < maxn; k++) {
      vis[i * prime[k]] = 1;
      if (i % prime[k] == 0) {
        d[i * prime[k]] = d[i] / (num[i] + 1) * (num[i] + 2);
        num[i * prime[k]] = num[i] + 1;
        break;
      } else {
        d[i * prime[k]] = d[i] * d[prime[k]];
        num[i * prime[k]] = 1;
      }
    }
  }
}
```

### 线性筛因数和
```cpp
const int maxn = 1e7 + 5;
bool vis[maxn];
int prime[maxn], sd[maxn], num[maxn], tot;
// sd[i]: i的约数和
void init() {
  memset(vis, 0, sizeof(vis));
  sd[1] = 1;
  for (ll i = 2; i < maxn; i++) {
    if (!vis[i]) {
      prime[tot++] = i;
      sd[i] = num[i] = 1 + i;
    }
    for (ll k = 0; k < tot && i * prime[k] < maxn; k++) {
      vis[i * prime[k]] = 1;
      if (i % prime[k]) {
        sd[i * prime[k]] = sd[i] * sd[prime[k]];
        num[i * prime[k]] = prime[k] + 1;
      } else {
        sd[i * prime[k]] = sd[i] / num[i] * (num[i] * prime[k] + 1);
        num[i * prime[k]] = num[i] * prime[k] + 1;
        break;
      }
    }
  }
}
```

### 欧拉函数

```cpp
// 求单个Eular函数
map<ll, ll> Eular;  //记忆化
ll eular(ll n) {
    ll &ret = Eular[n];
    if (ret) return ret;
    ret = n;
    for (ll i = 2; i * i <= n; i++) {
        if (n % i == 0) {
            ret -= ret / i;
            while (n % i == 0) n /= i;
        }
    }
    if (n > 1) ret -= ret / n;
    return ret;
}

// 线性筛 (同时得到欧拉函数和素数表)
const int maxn = 1e7 + 5;
bool vis[maxn];
int prime[maxn], phi[maxn];
void init() {
    clr(vis, 0);
    phi[1] = 1;
    int tot = 0;
    for (int i = 2; i < maxn; i++) {
        if (!vis[i]) {
            prime[tot++] = i;
            phi[i] = i - 1;
        }
        for (int k = 0; k < tot && 1LL * i * prime[k] < maxn; k++) {
            vis[i * prime[k]] = 1;
            if (i % prime[k] == 0) {
                phi[i * prime[k]] = phi[i] * prime[k];
                break;
            }
            phi[i * prime[k]] = phi[i] * (prime[k] - 1);
        }
    }
}
```
### 莫比乌斯函数
```cpp
const int maxn = 1e7 + 5;
bool check[maxn];
int mu[maxn];
vector<int> prime;
void CalMu() {
    mu[1] = 1;
    for (int i = 2; i < maxn; i++) {
        if (!check[i]) prime.push_back(i), mu[i] = -1;
        for (int j = 0; j < prime.size(); j++) {
            ll nxt = 1ll * prime[j] * i;
            if (nxt >= maxn) break;
            check[nxt] = true;
            if (i % prime[j] == 0) {
                mu[nxt] = 0;
                break;
            } else
                mu[nxt] = -mu[i];
        }
    }
}
```

### 高斯消元

```cpp
typedef double mat[maxn][maxn];
// A为增广矩阵，执行后A[i][n]表示第i个未知数的值
//无解或无穷多解返回false
bool gauss(mat &A, int n) {
    for (int i = 0; i < n; i++) {
        int r = i;
        for (int j = i + 1; j < n; j++)
            if (fabs(A[j][i]) > fabs(A[r][i])) r = j;
        if (r != i)
            for (int j = 0; j <= n; j++) swap(A[r][j], A[i][j]);
        if (fabs(A[i][i]) < eps) return false;

        for (int k = 0; k < n; k++)
            if (k != i)
                for (int j = n; k >= i; j--)
                    A[k][j] -= A[k][i] / A[i][i] * A[i][j];
    }
    return true;
}
// 模意义下用逆元代替除法
bool gauss(mat &A, int n) {
    for (int i = 0; i < n; i++) {
        int r = i;
        for (int j = i + 1; j < n; j++)
            if (fabs(A[j][i]) > fabs(A[r][i])) r = j;
        if (r != i)
            for (int j = 0; j <= n; j++) swap(A[r][j], A[i][j]);
        if (fabs(A[i][i] - eps) < eps) return false;

        ll b = Pow(A[i][i], mod - 2);
        for (int j = i; j < n + 1; j++) A[i][j] = A[i][j] * b % mod;
        for (int j = 0; j < n; j++)
            if (j != i) {
                int c = A[j][i];
                for (int k = i; k < n + 1; k++)
                    A[j][k] = (A[j][k] - A[i][k] * c % mod + mod) % mod;
            }
    }
    return true;
}
```

### 莫比乌斯函数

```cpp
// 线性筛莫比乌斯函数
const int maxn = 1e7;
int prime[maxn], tot, mu[maxn];
bool check[maxn];
void CalMu() {
    mu[1] = 1;
    for (int i = 2; i < maxn; i++) {
        if (!check[i]) prime[tot++] = i, mu[i] = -1;
        for (int j = 0; j < tot; j++) {
            if (i * prime[j] >= maxn) break;
            check[i * prime[j]] = true;
            if (i % prime[j] == 0) {
                mu[i * prime[j]] = 0;
                break;
            } else
                mu[i * prime[j]] = -mu[i];
        }
    }
}
```

### 整除分块

```cpp
// O(sqrt(n))
for (int l = 1, r; l <= n; l = r + 1) {
    r = n / (n / l);
    ans += (n / l) * (r - l + 1);
}
```

## 图论

### DFS 序

```cpp
// u 的子树区间 [in[u], out[u]]
// 下标从1开始
const int maxn = 100;
vector<int> G[maxn];
int in[maxn], out[maxn];
int dfn;

void init() { dfn = 0; }
void dfs(int u, int fa) {
    in[u] = ++dfn;
    for (auto &v : G[u]) {
        if (v != fa) dfs(v, u);
    }
    out[u] = dfn;
}
```

### 割点

```cpp
vector<int> G[maxn];
int dfs_clock, dfn[maxn];
bool iscut[maxn];
int dfs(int u, int fa) {
    int lowu = dfn[u] = ++dfs_clock;
    int child = 0;
    for (auto &v : G[u]) {
        if (!dfn[v]) {
            child++;
            int lowv = dfs(v, u);
            lowu = min(lowu, lowv);
            if (lowv >= dfn[u]) iscut[u] = true;
        } else if (dfn[v] < dfn[u] && v != fa)
            lowu = min(lowu, dfn[v]);
    }
    if (fa < 0 && child == 1) iscut[u] = false;  // root
    return lowu;
}
```

### 割边

```cpp
vector<pii> G[maxn];  // first: 下一个点， second: 该边的编号
int dfs_clock, dfn[maxn];
bool iscut[N];  // N: 边数

int dfs(int u, int fa) {
    int lowu = dfn[u] = ++dfs_clock;
    // int father = 0;
    for (auto &V : G[u]) {
        int v = V.first;
        int id = V.second;  // 边的编号
        // if(v == fa) father++;
        if (!dfn[v]) {
            int lowv = dfs(v, u);
            lowu = min(lowv, lowu);
            if (lowv > dfn[u]) iscut[id] = true;
        } else if (dfn[v] < dfn[u] && v != fa)
            lowu = min(lowu, dfn[v]);
    }
    // if(father > 1) return dfn[fa];
    return lowu;
}
```

### 点双联通分量

```cpp
vector<int> G[maxn], bcc[maxn];
int bcc_cnt, dfs_clock;
int dfn[maxn], iscut[maxn], bccno[maxn];
stack<pii> stk;
int dfs(int u) {
    int lowu = dfn[u] = ++dfs_clock;
    int child = 0;
    for (auto &v : G[u]) {
        pii e = {u, v};
        if (!dfn[v]) {
            stk.push(e);
            child++;
            int lowv = dfs(v, u);
            lowu = min(lowu, lowv);
            if (lowv >= dfn[u]) {  // u为割点
                iscut[u] = true;
                bcc_cnt++;
                bcc[bcc_cnt].clear();  //注意！bcc从1开始编号
                for (;;) {
                    pii x = stk.top();
                    stk.pop();
                    if (bccno[x.first] != bcc_cnt)
                        bcc[bcc_cnt].push_back(x.first), bcc[x.first] = bcc_cnt;
                    if (bccno[x.second] != bcc_cnt)
                        bcc[bcc_cnt].push_back(x.second),
                            bcc[x.second] = bcc_cnt;
                    if (x.first == u && x.second == v) break;
                }
            }
        } else if (dfn[v] < dfn[u] && v != fa) {
            stk.push(e);
            lowu = min(lowu, dfn[v]);
        }
    }
    if (fa < 0 && child == 1) iscut[u] = 0;
    return lowu;
}
//割顶的bccno无意义
void solve(int n) {
    //调用结束后stack保证为空，所以不用清空
    memset(iscut, 0, sizeof(iscut));
    memset(bccno, 0, sizeof(bccno));
    memset(dfn, 0, sizeof(dfn));
    dfs_clock = bcc_cnt = 0;
    for (int i = 0; i < n; i++)
        if (!dfn[i]) dfs(i, -1);
}
```

### 边双联通分量

```cpp
vector<int> G[maxn];
int bcc_cnt, dfs_clock;
int dfn[maxn], bccno[maxn];
stack<int> stk;
int dfs(int u, int fa) {
    int lowu = dfn[u] = ++dfs_clock;
    stk.push(u);
    bool flag = false;  // 有重边
    for (auto &v : G[u]) {
        if (v == fa && !flag) {
            flag = true;
            continue;
        }
        if (!dfn[v]) {
            int lowv = dfs(v, u);
            lowu = min(lowu, lowv);
        } else
            lowu = min(lowu, dfn[v]);
    }
    if (lowu == dfn[u]) {
        bcc_cnt++;  // 编号从1开始
        while (!stk.empty()) {
            int v = stk.top();
            bccno[v] = bcc_cnt;
            if (u == v) break;
        }
    }
}
```

### 强连通分量

```cpp
vector<int> G[maxn];
int scc, dfs_clock, top;  // scc: 强连通分量的数量
bool instack[maxn];
int dfn[maxn], low[maxn], belong[maxn], Stack[maxn];
// int num[maxn];      // 每个强连通分量的数量。 1 ~ scc
// int maps[maxn];      //缩点之后 每个点对应的新点的标号
void Tarjan(int u) {
    dfn[u] = low[u] = ++dfs_clock;
    instack[u] = true;
    Stack[top++] = u;
    for (auto &v : G[u]) {
        if (!dfn[v]) {
            Tarjan(v);
            low[u] = min(low[u], low[v]);
        } else if (instack[v])
            low[u] = min(low[u], dfn[v]);
    }
    if (dfn[u] == low[u]) {
        ++scc;
        int cnt = 0;
        int now;
        while (top > 0) {
            now = Stack[--top];
            instack[now] = false;
            belong[now] = u;
            ++cnt;
            if (now == u) {
                // num[scc] = cnt;
                // maps[u] = scc;
                break;
            }
        }
    }
}
void solve(int n) {
    memset(instack, 0, sizeof(instack));
    memset(dfn, 0, sizeof(dfn));
    scc = dfs_clock = top = 0;
    for (int i = 1; i <= n; i++) {  // 点的标号从1开始
        if (!dfn[i]) Tarjan(i);
    }
}
```

### 单源最短路

#### Dijkstra

```cpp
// O(ElogV)
typedef pair<int, int> pii;
const int maxn = 100;
int d[maxn];
vector<pii> G[maxn];

inline void addedge(int u, int v, int w) {
    G[u].push_back(make_pair(w, v));
    // G[v].push_back(make_pair(w,u));
}
void dijkstra(int s, int n) {
    for (int i = 0; i < n; i++) d[i] = INF;
    d[s] = 0;
    priority_queue<pii, vector<pii>, greater<pii> > pq;
    pq.push(mp(0, s));
    while (!pq.empty()) {
        pii now = pq.top();
        pq.pop();
        int v = now.second;
        if (d[v] < now.first) continue;  //剪枝！ 重要
        for (int i = 0; i < G[v].size(); i++) {
            pii e = G[v][i];
            int to = e.second;
            if (d[to] > d[v] + e.first) {
                d[to] = d[v] + e.first;
                pq.push(pii(d[to], to));
            }
        }
    }
}
```

#### Bellman-Ford

### 最小生成树

#### Kruskal

```cpp
const int maxn = 10;
struct Edge {
    int u, v, w;
    Edge(int u = 0, int v = 0, int w = 0) : u(u), v(v), w(w) {}
    bool operator<(const Edge &A) const { return w < A.w; }
};
vector<Edge> edges;
int par[maxn];
void addedge(int u, int v, int w) {
    edges.push_back(Edge(u, v, w));
    // edges.push_back(Edge(v, u, w));
}

int find(int x) { return par[x] == x ? x : find(par[x]); }

int kruskal(int n) {
    int ans = 0;
    for (int i = 0; i < n; i++) par[i] = i;
    sort(edges.begin(), edges.end());  //按权值排序
    int cnt = 0;
    for (int i = 0; i < edges.size(); i++) {
        if (cnt >= n - 1) break;
        Edge &now = edges[i];
        int x = find(now.u);
        int y = find(now.v);
        if (x != y) {
            cnt++;
            par[x] = y;
            ans += now.w;
        }
    }
    return ans;
}
```

#### Prim

```cpp
// 耗费矩阵cost[][],标号从0开始,0~n-1
// 返回最小生成树的权值,返回-1表示原图不连通
// O(n^2)
const int maxn = 100;
bool vis[maxn];
int lowc[maxn];
int cost[maxn][maxn];  //初始化为正无穷

int Prim(int n) {
    int ans = 0;
    clr(vis, 0);
    vis[0] = 1;
    for (int i = 1; i < n; i++) lowc[i] = cost[0][i];
    for (int i = 1; i < n; i++) {
        int minc = INF;
        int p = -1;
        for (int j = 0; j < n; j++)
            if (!vis[j] && minc > lowc[j]) {
                minc = lowc[j];
                p = j;
            }
        if (minc == INF) return -1;
        vis[p] = 1;
        ans += minc;
        for (int j = 0; j < n; j++)
            if (!vis[j] && lowc[j] > cost[p][j]) lowc[j] = cost[p][j];
    }
    return ans;
}
```

### 最近公共祖先 LCA

#### ST

```cpp
// 在线算法，预处理O(nlogn)，查询O(1)
const int maxn = 1000000;
struct LCA {
    vector<int> G[maxn], sp;
    int dep[maxn], dfn[maxn];
    pii dp[21][maxn << 1];
    void init(int n) {
        for (int i = 0; i <= n; i++) G[i].clear();
        sp.clear();
    }
    void addEdge(int u, int v) {
        G[u].push_back(v);
        G[v].push_back(u);
    }
    void dfs(int u, int fa) {
        dep[u] = dep[fa] + 1;
        dfn[u] = sp.size();
        sp.push_back(u);
        for (auto &v : G[u]) {
            if (v == fa) continue;
            dfs(v, u);
            sp.push_back(u);
        }
    }
    void initRmq() {
        int n = sp.size();
        for (int i = 0; i < n; i++) dp[0][i] = {dfn[sp[i]], sp[i]};
        for (int i = 1; (1 << i) <= n; i++)
            for (int j = 0; j + (1 << i) - 1 < n; j++)
                dp[i][j] = min(dp[i - 1][j], dp[i - 1][j + (1 << (i - 1))]);
    }
    int lca(int u, int v) {
        int l = dfn[u], r = dfn[v];
        if (l > r) swap(l, r);
        int k = 31 - __builtin_clz(r - l + 1);
        return min(dp[k][l], dp[k][r - (1 << k) + 1]).second;
    }
} lca;
```

#### Tarjan

```cpp
// 离线算法 O(n+q)
const int maxn = 1000000;
struct LCA {
    vector<int> G[maxn];
    vector<pii> Q[maxn];
    bool vis[maxn];
    int par[maxn], dist[maxn], ANS[maxn];

    int find(int u) { return u == par[u] ? u : par[u] = find(par[u]); }
    void merge(int u, int v) {
        u = find(u);
        v = find(v);
        if (u != v) {
            par[v] = u;
        }
    }

    void init(int n, int m) {
        for (int i = 0; i < n + 1; i++) {
            G[i].clear();
            vis[i] = false;
            par[i] = i;
        }
        for (int i = 0; i < m + 1; i++) Q[i].clear();
    }

    void add_edge(int u, int v) {
        G[u].pb(v);
        G[v].pb(u);
    }

    void add_query(int u, int v, int id) {
        Q[u].pb(mp(v, id));
        Q[v].pb(mp(u, id));
    }

    void Tarjan(int u, int fa) {
        vis[u] = true;
        for (auto v : G[u]) {
            if (fa != v) {
                Tarjan(v, u);
                merge(u, v);  // 注意合并顺序，一定是v合并到u上
            }
        }
        for (auto V : Q[u]) {
            if (vis[V.first] == true) {
                ANS[V.second] = find(V.first);  // 公共祖先为par[v]
            }
        }
    }
} lca;
```

### Dsu-on-tree

```cpp
// O(nlogn)
// 统计所有子树的信息。启发式合并，保留重儿子的信息。
// 给一棵树，每个节点都有一个颜色，每次查询问某棵子树上颜色为c的节点数
vector<int> *vec[maxn], G[maxn];
int cnt[maxn], sz[maxn], col[maxn];
bool vis[maxn], bigc[maxn];

int szdfs(int u, int fa) {
    sz[u] = vis[u] = 1;
    for (auto &v : G[u])
        if (v != fa && !vis[v]) sz[u] += szdfs(v, u);
    return sz[u];
}

void add(int u, int fa, int x) {
    cnt[col[u]] += x;
    for (auto &v : G[u])
        if (v != fa && !bigc[v]) add(v, u, x);
}

void dfs(int u, int fa, bool keep) {
    int mx = -1, bc = -1;
    for (auto &v : G[u])
        if (v != fa && sz[v] > mx) {
            mx = sz[v];
            bc = v;
        }
    for (auto &v : G[u])
        if (v != fa && v != bc)
            dfs(v, u, 0);  // run a dfs on small childs and clear them from cnt
    if (bc != -1) {
        dfs(bc, u, 1);
        bigc[bc] = 1;
    }  // bigChild marked as big and not cleared from cnt
    add(u, fa, 1);

    // now cnt[c] is the number of vertices in subtree of vertex v that has
    // color c. You can answer the queries easily.
    if (bc != -1) bigc[bc] = 0;
    if (!keep) add(u, fa, -1);
}
```

### 网络流

#### 最大流

```cpp
struct Edge{
    int from,to,cap,flow;
    Edge(int u,int v,int c,int f): from(u),to(v),cap(c),flow(f) {}
};
struct EdmonsKarp{          //时间复杂度O(v*E*E)
    int n,m;
    vector<Edge> edges;     //边数的两倍
    vector<int> G[maxn];    //邻接表，G[i][j]表示节点i的第j条边在e数组中的序号
    int a[maxn];            //起点到i的可改进量
    int p[maxn];            //最短路树上p的入弧编号

    void init(int n){
        for(int i=0;i<=n;i++) G[i].clear();
        edges.clear();
    }

    void AddEdge(int from,int to,int cap){
        edges.push_back(Edge(from,to,cap,0));
        edges.push_back(Edge(to,from,0,0));     //反向弧
        m = edges.size();
        G[from].push_back(m-2);
        G[to].push_back(m-1);
    }

    int Maxflow(int s,int t){
        int flow = 0;
        for(;;){
            memset(a,0,sizeof(a));
            queue<int> Q;
            Q.push(s);
            a[s] = INF;
            while(!Q.empty()){
                int x = Q.front();Q.pop();
                for(int i=0;i<G[x].size();i++){
                    Edge& e = edges[G[x][i]];
                    if(!a[e.to] && e.cap > e.flow){     //!a[e.to] 是了保证不会回退和出现分叉
                        p[e.to] = G[x][i];      //记录边的编号
                        a[e.to] = min(a[x],e.cap - e.flow);
                        Q.push(e.to);
                    }
                }
                if(a[t]) break;         //走到汇点
            }
            if(!a[t]) break;            //没有一条增广路存在
            for(int u=t;u != s;u = edges[p[u]].from){
                edges[p[u]].flow += a[t];
                edges[p[u]^1].flow -= a[t];
            }
            flow += a[t];
        }
        return flow;
    }
};
```

#### 最小费用最大流

```cpp
struct Edge{
    int from,to,cap,flow,cost;
    Edge(int u,int v,int c,int f,int w):from(u),to(v),cap(c),flow(f),cost(w) {}
};

struct MCMF{
    int n,m;
    vector<Edge> edges;
    vector<int> G[maxn];
    int inq[maxn];      //是否在队列中
    int d[maxn];        //bellmanford 到源点距离
    int p[maxn];        //上一条弧
    int a[maxn];        //可改进量

    void init(int n){
        this-> n = n;
        for(int i=0;i<=n;i++) G[i].clear();
        edges.clear();
    }

    void AddEdge(int from,int to,int cap,int cost){
        edges.push_back(Edge(from,to,cap,0,cost));
        edges.push_back(Edge(to,from,0,0,-cost));
        m = edges.size();
        G[from].push_back(m-2);
        G[to].push_back(m-1);
    }

    bool BellmanFord(int s,int t,int& flow,ll& cost){
        for(int i=0;i<=n;i++) d[i] = INF;
        memset(inq,0,sizeof(inq));
        d[s] = 0; inq[s] = 1; p[s] = 0; a[s] = INF;
        queue<int> Q;
        Q.push(s);
        while(!Q.empty()){
            int u = Q.front(); Q.pop();
            inq[u] = 0;
            for(int i=0;i<G[u].size();i++){
                Edge& e = edges[G[u][i]];
                if(e.cap > e.flow && d[e.to] > d[u] + e.cost){
                    d[e.to] = d[u] + e.cost;
                    p[e.to] = G[u][i];
                    a[e.to] = min(a[u], e.cap - e.flow);
                    if(!inq[e.to]) {Q.push(e.to); inq[e.to] = 1;}
                }
            }
        }
        if(d[t] == INF) return false;    // 当没有可增广的路时退出
        flow += a[t];
        cost += (ll)d[t] * (ll)a[t];
        for(int u=t; u!= s; u = edges[p[u]].from){
            edges[p[u]].flow += a[t];
            edges[p[u]^1].flow -= a[t];
        }
        return true;
    }

    int MincostMaxflow(int s,int t,ll& cost){
        int flow = 0; cost = 0;
        while(BellmanFord(s,t,flow,cost));
        return flow;
    }
};
```

## 数据结构

### 树状数组

```cpp
// 数组下标必须从1开始

// 单点更新区间查询
int BIT[maxn];
inline int lowb(int x) { return x & (-x); }
inline int query(int l,int r){
    int ret = 0;
    for(int i=l-1;i>0;i-=lowb(i)) ret -= BIT[i];
    for(int i=r;i>0;i-=lowb(i)) ret += BIT[i];
    return ret;
}
inline void update(int x,int y,int n){
    for(int i=x;i<=n;i+=lowb(i)) BIT[i] += y;
}
//区间更新单点查询
int diff[maxn];
inline int lowb(int x) {return x &(-x);}
inline int query(int x){
    int ret = 0;
    for(int i=x;i>0;i-=lowb(i)) ret += diff[i];
    return ret;
}
inline void update(int l,int r,int val,int n){
    for(int i=l;i<=n;i+=lowb(i)) diff[i] += val;
    for(int i=r+1;i<=n;i+=lowb(i)) diff[i] -= val;
}
```

### 线段树

```cpp
// 左闭右闭
#define lson (rt << 1)
#define rson (rt << 1 | 1)
#define lson_len (len - (len >> 1))
#define rson_len (len >> 1)
const int maxn = "Edit";
int seg[maxn << 2];
```

**单点更新，区间查询**

```cpp
// 左闭右闭 [l,r]
void pushup(int rt) { seg[rt] = seg[lson] + seg[rson]; }

void build(int l, int r, int rt) {
    if (l == r) {
        seg[rt] = num[l];
        // cin >> seg[rt];   //建立的时候直接输入叶节点
        return;
    }
    int m = (l + r) >> 1;
    build(l,m,lson);
    build(m+1,r,rson);
    pushup(rt);
}

void update(int p, int add, int l, int r, int rt) {
    if (l == r) {
        seg[rt] += add;
        return;
    }
    int m = (l + r) >> 1;
    if (p <= m) update(p, add,l,m,lson);
    else update(p, add, m+1,r,rson);
    pushup(rt);
}
int query(int L, int R, int l, int r, int rt) {  // L R 是要查询的区间
    if (L <= l && r <= R) return seg[rt];
    int m = (l + r) >> 1, ret = 0;
    if (L <= m) s += query(L, R, l,m,lson);
    if (m < R) s += query(L, R, m+1,r,rson);
    return ret;
}

// update interval
// lazy[rt]用于存放懒惰标记，注意PushDown时标记的传递
const int maxn = "Edit";
int lazy[maxn << 2], seg[maxn << 2];

void pushup(int rt) { seg[rt] = seg[lson] + seg[rson]; }
void pushdown(int rt, int len) {
    if (lazy[rt] == 0) return;
    lazy[lson] += lazy[rt];
    lazy[rson] += lazy[rt];
    seg[lson] += lazy[rt] * lson_len;
    seg[rson] += lazy[rt] * rson_len;
    lazy[rt] = 0;
}
void build(int l, int r, int rt) {
    lazy[rt] = 0;
    if (l == r) {
        seg[rt] = num[l];
        //cin >> seg[rt];
        return;
    }
    int m = (l + r) >> 1;
    build(l,m,lson);
    build(m+1,r,rson);
    pushup(rt);
}
void update(int L, int R, int add, int l, int r, int rt) {    // L R 是要更新的区间
    if (L <= l && r <= R) {
        lazy[rt] += add;
        seg[rt] += add * (r - l + 1);
        return;
    }
    pushdown(rt, r - l + 1);
    int m = (l + r) >> 1;
    if (L <= m) update(L, R, add, l,m,lson);
    if (m < R) update(L, R, add, m+1,r,rson);
    pushup(rt);
}
int query(int L, int R, int l, int r, int rt) {
    if (L <= l && r <= R) return seg[rt];
    pushdown(rt, r - l + 1);
    int m = (l + r) >> 1, ret = 0;
    if (L <= m) ret += query(L, R, l,m,lson);
    if (m < R) ret += query(L, R, m+1,r,rson);
    return ret;
}

// interval merge
struct Seg{
    int mx,lmx,rmx;
};

void pushup(int rt,int len){
    seg[rt].mx = max(seg[lson].mx, max(seg[rson].mx, seg[lson].rmx + seg[rson].lmx));
    seg[rt].lmx = seg[lson].lmx;
    seg[rt].rmx = seg[rson].rmx;
    if(seg[rt].lmx == lson_len) seg[rt].lmx += seg[rson].lmx;
    if(seg[rt].rmx == rson_len) seg[rt].rmx += seg[lson].rmx;
}
```

### 单调栈

```cpp
int L[maxn], R[maxn];
int a[maxn];

// 找到左边第一个大于它的元素位置l，和右边第一个大于它的元素位置r
// [l,r]
void init(int n) {
    stack<int> st;
    for (int i = 0; i < n; i++) {
        while (!st.empty() && a[i] > a[st.top()]) st.pop();
        if (st.empty())
            L[i] = 0;
        else
            L[i] = st.top() + 1;
        st.push(i);
    }
    while (!st.empty()) st.pop();
    for (int i = n - 1; i >= 0; i--) {
        while (!st.empty() && a[i] > a[st.top()]) st.pop();
        if (st.empty())
            R[i] = n-1;
        else
            R[i] = st.top() - 1;
        st.push(i);
    }
}
```

### 单调队列

```cpp
deque<pii> dq;  // (pos, val)

for(int i=1;i<=n;i++){
    while(dq.size() && 队首过期) dq.pop_front();
    while(dq.size() && 加入当前元素后队列不单调) dq.pop_back();
    dq.push_back(当前元素)
    队首进行决策
}
```

### 树链剖分

```cpp
const int maxn = 1000000;
struct HLD {
    int n, dfn;
    int sz[maxn], top[maxn], son[maxn], dep[maxn], par[maxn], id[maxn],rk[maxn];
    vector<int> G[maxn];
    void init(int n) {
        this->n = n;
        clr(son, -1);
        dfn = 0;
        for (int i = 0; i <= n; i++) G[i].clear();
    }
    void addedge(int u,int v){
        G[u].pb(v);
        G[v].pb(u);
    }
     void build() {
        dfs(1, 1, 1);
        link(1, 1);
    }
    void dfs(int u,int fa,int d){
        dep[u] = d;
        par[u] = fa;
        sz[u] = 1;
        for(auto &v:G[u]){
            if(v == fa) continue;
            dfs(v,u,d+1);
            sz[u] += sz[v];
            if(son[u] == -1 || sz[v] > sz[son[u]]) son[u] = v;
        }
    }
    void link(int u,int t){
        top[u] = t;
        id[u] = ++dfn;
        rk[dfn] = u;
        if(son[u] == -1) return ;
        link(son[u],t);     // 保证重链的dfs序是连续的
        for(auto &v:G[u]){
            if(v != son[u] && v != par[u])
                link(v,v);
        }
    }

    void update(int u,int v,int w) {} // 数据结构
    int query(int u,int v) {}         // 数据结构

    void update_path(int u,int v,int w){
        while(top[u] != top[v]){
            if(dep[top[u]] < dep[top[v]]) swap(u,v);
            update(id[top[u]],id[u],w);
            u = par[top[u]];
        }
        // if(u == v) return;   // 边权
        if(dep[u] > dep[v]) swap(u,v);
        update(id[u],id[v],w);         // 点权
        // update(id[u] + 1,id[v],w);  // 边权

    }
    int query_path(int u,int v){
        int ret = 0;
        while(top[u] != top[v]){
            if(dep[top[u]] < dep[top[v]]) swap(u,v);
            ret += query(id[top[u]],id[u]);
            u = par[top[u]];
        }
        // if(u == v) return ret;   // 边权
        if(dep[u] > dep[v]) swap(u,v);
        ret += query(id[u],id[v]);         // 点权
        // ret += query(id[u] + 1,id[v]);  // 边权
        return ret;
    }
} hld;
```

### 主席树

```cpp
const int maxn = 100;
const int N = maxn + Qlog(n);
int root[maxn];     // 记录每个版本的根节点
int sum[N], lch[N], rch[N];
int dfn = 0;

// 单点修改，区间查询
inline void pushup(int rt) { sum[rt] = sum[lch[rt]] + sum[rch[rt]]; }
void build(int &k, int l, int r) {
    k = ++dfn;
    if (l == r) {
        sum[k] = a[l];
        return;
    }
    int m = l + r >> 1;
    build(lch[k], l, m);
    build(rch[k], m + 1, r);
    pushup(k);
}

void newnode(int old, int k) {
    lch[k] = lch[old];
    rch[k] = rch[old];
    sum[k] = sum[old];
}
void update(int old, int &k, int l, int r, int p, int x) {
    k = ++dfn;
    newnode(old, k);
    if (l == r) {
        sum[k] += x;
        return;
    }
    int m = l + r >> 1;
    if (p <= m) update(lch[old], lch[k], l, m, p, x);
    if (p > m) update(rch[old], rch[k], m + 1, r, p, x);
    pushup(k);
}

int query(int k, int l, int r, int L, int R) {
    if (L <= l && R >= r) return sum[k];
    int m = l + r >> 1;
    int ret = 0;
    if(m >= L) ret += query(lch[k],l,m,L,R);
    if(m < R) ret += query(rch[k],m+1,r,L,R);
    return ret;
}

// 区间更新 区间查询
int lazy[N];

inline void pushup(int rt, int len) {
    sum[rt] = sum[lch[rt]] + sum[rch[rt]] + len * lazy[sum];
}
void build(int &k, int l, int r) {
    k = ++dfn;
    lazy[k] = 0;
    if (l == r) {
        cin >> sum[k];
        return;
    }
    int m = l + r >> 1;
    build(lch[k], l, m);
    build(rch[k], m + 1, r);
    pushup(k, r - l + 1);
}

inline void newnode(int old, int k) {
    lch[k] = lch[old];
    rch[k] = rch[old];
    sum[k] = sum[old];
    lazy[k] = lazy[old];
}
void update(int old, int &k, int l, int r, int L, int R, int x) {
    k = ++dfn;
    newnode(old, k);
    if (L <= l && R >= r) {
        sum[k] += x * (r - l + 1);
        lazy[k] += x;
        return;
    }
    int m = l + r >> 1;
    if (m >= L) update(lch[old], lch[k], l, m, L, R, x);
    if (m < R) update(rch[old], rch[k], m + 1, r, L, R, x);
    pushup(k, r - l + 1);
}

int query(int k, int l, int r, int L, int R, int x) {
    if (L <= l && R >= r) return sum[k] + x * (r - l + 1);
    x += lazy[k];
    int m = l + r >> 1;
    int ret = 0;
    if(m >= L) ret += query(lch[k],l,m,L,R,x);
    if(m < R) ret += query(rch[k],m+1,r,L,R,x);
    return ret;
}
```

## 动态规划

### RMQ

#### 一维 RMQ

```cpp
// nlog(n) 预处理， log(n) 查询
// 起始下标为1
const int maxn = 100;
int dp[maxn][maxn];
int a[maxn];
void rmq_init(int n) {
    for (int i = 1; i <= n; i++) dp[i][0] = a[i];
    for (int k = 1; (1 << k) <= n; k++) {
        for (int i = 1; i + (1 << k) < n; i++) {
            dp[i][k] = min(dp[i][k - 1], dp[i + (1 << (k - 1))][k - 1]);
        }
    }
}
int rmq(int l, int r) {
    int k = 31 - __builtin_clz(r - l + 1);  // 前导零的个数
    return min(d[l][k], d[r - (1 << k) + 1][k]);
}
```

#### 二维 RMQ

```cpp
// dp[r][c][i][k]: 表示(r,c)为左上角,(r + 2^i - 1, c + 2^k - 1)为右下角的矩阵中的最小值
// 预处理 n*m*log(n)*log(m)， 查询 log(nm)
// 起始下标为1
const int maxn = 100;
int dp[maxn][maxn][maxn][maxn];
int a[maxn][maxn];
void rmq_init(int n, int m) {
    for (int i = 1; i <= n; i++) {
        for (int k = 1; k <= m; k++) {
            dp[i][k][0][0] = a[i][k];
        }
    }
    for (int i = 0; (1 << i) <= n; i++) {
        for (int k = 0; (1 << k) <= m; k++) {
            if (!i && !k) continue;
            for (int r = 1; r + (1 << i) - 1 <= n; r++) {
                for (int c = 1; c + (1 << k) - 1 <= m; c++) {
                    if (!k)
                        dp[r][c][i][k] =
                            min(dp[r][c][i - 1][k],
                                dp[r + (1 << (i - 1))][c][i - 1][k]);
                    else
                        dp[r][c][i][k] =
                            min(dp[r][c][i][k - 1],
                                dp[r][c + (1 << (k - 1))][i][k - 1]);
                }
            }
        }
    }
}
int rmq(int x1, int y1,int x2,int y2) {
    int kx = 31 - __builtin_clz(x2 - x1 + 1);
    int ky = 31 - __builtin_clz(y2 - y1 + 1);
    int m1 = dp[x1][y1][kx][ky];
    int m2 = dp[x2 - (1 << kx) + 1][y1][kx][ky];
    int m3 = dp[x1][y2 - (1 << ky) + 1][kx][ky];
    int m4 = dp[x2 - (1 << kx) + 1][y2 - (1 << ky) + 1][kx][ky];
    return min({m1,m2,m3,m4});
}
```

### 子集 DP

```cpp
// O(3^n)
for (int i = 1; i < (1 << n); i++) {
    for (int j = i; j; j = (j - 1) & i) {
        /*
            j为i的子集
        */
    }
}
```

```cpp
// O(n2^n)
for (int i = 0; i < n; i++) {
    for (int j = 0; j < (1 << n); j++) {
        if (j & (1 << i)) {
        /*
            j ^ (1 << i) 为 j 的子集
        */
        }
    }
}
```

## 博弈

### SG 函数

```cpp
// sg[] = 0 必败，其余必胜
const int maxn = 100;
int f[N];
int SG[maxn];  //
int S[maxn];   // x 的后继状态

void getSG(int n) {
    SG[0] = 0;  // clr(SG,0);
    for (int i = 1; i < maxn; i++) {
        clr(S, 0);
        for (int k = 0; f[k] <= i && k < n; k++) S[SG[i - f[k]]] = 1;
        for (int k = 0;; k++) {
            if (!S[k]) {
                SG[i] = k;
                break;
            }
        }
    }
}

// 单点SG查询
void init(){
    clr(SG,-1);
}
int getSG(int n, int foo) {
    if (SG[foo] != -1) return SG[foo];
    // int S[N] = {0};
    set<int> S;
    for (int k = 0; f[k] <= foo && k < n; k++) {
        int bar = getSG(n, foo - f[k]);
        S.insert(bar);
        // S[bar] = 1;
    }
    for (int k = 0;; k++) {
        if (S.find(k) == S.end()) {
            return SG[foo] = k;
        }
    }
}
```

## 其它

### 三分

#### 整数三分

```cpp
// 求极大值
int find(int l, int r) {
    while (l < r - 1) {
        int m1 = l + r >> 1;
        int m2 = m1 + r >> 1;
        if (work(m1) > work(m2))
            r = m2;
        else
            l = m1;
    }
    return work(l) > work(r) ? l : r;
}
```

#### 浮点三分

```cpp
// 求极小值
double find(double l, double r) {
    double m1, m2;
    while (r - l >= eps) {
        m1 = l + (r - l) / 3;
        m2 = r - (r - l) / 3;
        // m1 = l + r >> 1;
        // m2 = m1 + r >> 1;
        if (work(m1) > work(m2))
            l = m1;
        else
            r = m2;
    }
    return (m1 + m2) / 2;
}
```

### 三维前缀和

```cpp
int a[maxn][maxn][maxn];

int sum(int x1, int y1, int z1, int x2, int y2, int z2) {
    return a[x2][y2][z2] - a[x1 - 1][y2][z2] - a[x2][y1 - 1][z2] -
           a[x2][y2][z1 - 1] + a[x1 - 1][y1 - 1][z2] + a[x1 - 1][y2][z1 - 1] +
           a[x2][y1 - 1][z1 - 1] - a[x1 - 1][y1 - 1][z1 - 1];
}

void init(int n) {
    for (int i = 0; i < n; i++)
        for (int k = 0; k < n; k++)
            for (int t = 0; t < n; t++) a[i][k][t] += a[i - 1][k][t];
    for (int i = 0; i < n; i++)
        for (int k = 0; k < n; k++)
            for (int t = 0; t < n; t++) a[i][k][t] += a[i][k - 1][t];
    for (int i = 0; i < n; i++)
        for (int k = 0; k < n; k++)
            for (int t = 0; t < n; t++) a[i][k][t] += a[i][k][t - 1];
}
```

### 离散化

```cpp
// c++11
vector<int> a,b;
for(int i=0;i<n;i++){
    cin >> a[i];
    b[i] = a[i];
}
b.sort(b.begin(),b.end());
b.resize(distance(b.begin(),unique(b.begin(),b.end())));

inline int getid(int x){
    return lower_bound(b.begin(),b.end(),x) - b.begin() + 1;
}
```

```cpp
for(int i=0;i<n;i++){
    A[i] = B[i];
}
sort(B,B+n);
int size = unique(B,B+n) - B;
for(int i=0;i<n;++){
    A[i] = lower_bound(B,B+size,A[i]) - B + 1;
}
```
