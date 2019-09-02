---
title: FFT
comments: true
date: 2018-08-26 17:10:41
categories:
- ACM
- 数学
- FFT
---

# FFT

## 基本形式

$h(n) = \sum_{i=0}^{n}f(i)g(n-i)$

表示多项式$f$和$g$相乘后，$n$次项的系数为$h(n)$。

| A[0]   | A[1]   | A[2]   | ... | A[n]   |
| ------ | ------ | ------ | --- | ------ |
| $f(0)$ | $f(1)$ | $f(2)$ | ... | $f(n)$ |

| B[0]   | B[1]   | B[2]   | ... | B[n]   |
| ------ | ------ | ------ | --- | ------ |
| $g(0)$ | $g(1)$ | $g(2)$ | ... | $g(n)$ |

$h(n) = C[n]$

## 变式

### 1

$h(n) = \sum_{i=k}^{n}f(i)g(n-i)$

| A[0]   | A[1]     | A[2]     | ... | A[n-k] |
| ------ | -------- | -------- | --- | ------ |
| $f(k)$ | $f(k+1)$ | $f(k+2)$ | ... | $f(n)$ |

| B[0]   | B[1]   | B[2]   | ... | B[n-k]   |
| ------ | ------ | ------ | --- | -------- |
| $g(0)$ | $g(1)$ | $g(2)$ | ... | $g(n-k)$ |

$h(n) = C[n-k]$

---

### 2

$h(n) = \sum_{i=0}^{n}f(i)g(i+a)$

设$z(n-i) = f(i)$

则原式变为$h(n+a) = \sum_{i=0}^{n}z(n-i)g(i+a)$

| A[0]   | A[1]   | A[2]   | ... | A[n]   |
| ------ | ------ | ------ | --- | ------ |
| $z(0)$ | $z(1)$ | $z(2)$ | ... | $z(n)$ |

| B[0]   | B[1]     | B[2]     | ... | B[n]     |
| ------ | -------- | -------- | --- | -------- |
| $g(a)$ | $g(a+1)$ | $g(a+2)$ | ... | $g(a+n)$ |

$h(n+a) = C[n]$

---

### 3

$h(n) = \sum_{i=k}^{n}f(i)g(i+a)$

设$z(n-i) = f(i)$

则原式变为$h(n+a) = \sum_{i=k}^{n}z(n-i)g(i+a)$

| A[0]   | A[1]   | A[2]   | ... | A[n-k]   |
| ------ | ------ | ------ | --- | -------- |
| $z(0)$ | $z(1)$ | $z(2)$ | ... | $z(n-k)$ |

| B[0]     | B[1]       | B[2]       | ... | B[n-k]   |
| -------- | ---------- | ---------- | --- | -------- |
| $g(k+a)$ | $g(k+a+1)$ | $g(k+a+2)$ | ... | $g(a+n)$ |

$h(n+a) = C[n-k]$

---

### 4
$h(n) = \sum_{i=1}^{n}\sum_{j=1}^{n}x^{ij}$ 其中 $x$ 为常数。

$$\begin{aligned}
h(n) &= \sum_{i=1}^{n}\sum_{j=1}^{n}x^{ij} \\\\
&= \sum_{i=1}^{n}\sum_{j=1}^{n}x^{\frac{1}{2}\times (i^2 + j^2 - (i-j)^2)} \\\\
&= \sum_{i=1}^{n}x^{\frac{1}{2} \times i^2}\sum_{j=1}^{n}x^{\frac{1}{2} \times j^2} x^{-\frac{1}{2} \times (i-j)^2} \\\\
&= \sum_{i=1}^{n}x^{\frac{1}{2} \times i^2}\sum_{j=1}^{n}f_jg_{i-j}
\end{aligned} 
$$

## 模板

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

vector<int> multiply(vector<int>& a, vector<int>& b) {
    int need = a.size() + b.size() - 1;
    int nbase = 0;
    while ((1 << nbase) < need) nbase++;
    ensure_base(nbase);
    int sz = 1 << nbase;
    if (sz > (int)fa.size()) fa.resize(static_cast<unsigned long>(sz));
    for (int i = 0; i < sz; i++) {
        int x = (i < (int)a.size() ? a[i] : 0);
        int y = (i < (int)b.size() ? b[i] : 0);
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
    vector<int> res(static_cast<unsigned long>(need));
    for (int i = 0; i < need; i++) {
        res[i] = fa[i].x + 0.5;
    }
    return res;
}
};  // namespace fft

```
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
#define first fi
#define second se
#define my_unique(a) a.resize(distance(a.begin(), unique(a.begin(), a.end())))
#define my_sort_unique(c) (sort(c.begin(), c.end())), my_unique(c)
typedef long long ll;
typedef pair<int, int> pii;
const int INF = 0x3f3f3f3f;
const int NINF = 0xc0c0c0c0;
const int maxn = 2e5 + 5;
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
/*
* 进行FFT和IFFT前的反转变换。
* 位置i和 （i二进制反转后位置）互换
* len必须取2的幂
*/
void change(Complex y[], int len) {
    for (int i = 1, j = len / 2; i < len - 1; i++) {
        if (i < j) swap(y[i], y[j]);
        //交换互为小标反转的元素，i<j保证交换一次
        // i做正常的+1，j左反转类型的+1,始终保持i和j是反转的
        int k = len / 2;
        while (j >= k) j -= k, k /= 2;
        if (j < k) j += k;
    }
}
/*
* 做FFT
* len必须为2^k形式，
* on==1时是DFT，on==-1时是IDFT
* DFT:系数表示法->点值表示法
* IDFT:点值表示法->系数表示法
*/
void fft(Complex y[], int len, int on) {
    change(y, len);
    for (int h = 2; h <= len; h <<= 1) {
        Complex wn(cos(-on * 2 * PI / h), sin(-on * 2 * PI / h));
        // 计算当前的单位复根
        for (int j = 0; j < len; j += h) {
            Complex w(1, 0);
            // 计算当前的单位复根
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
Complex x1[maxn], x2[maxn];
char a[maxn], b[maxn];
int sum[maxn];
int main() {
// /*
#ifndef ONLINE_JUDGE
    freopen("1.in", "r", stdin);
    freopen("1.out", "w", stdout);
#endif
    // */
    std::ios::sync_with_stdio(false);
    while (~scanf("%s%s", &a, &b)) {
        int len1 = strlen(a);
        int len2 = strlen(b);
        int len = 1;
        while (len < len1 * 2 || len < len2 * 2)
            len <<= 1;  // len > len1 + len2
        for(int i=0;i<len;i++){
            sum[i] = 0;
            x1[i] = Complex(0, 0);
            x2[i] = Complex(0, 0);
        }
        for (int i = 0; i < len1; i++) 
            x1[i] = Complex(a[i] - '0', 0);
        for (int i = 0; i < len2; i++)
            x2[i] = Complex(b[i] - '0', 0);
        fft(x1, len, 1);
        fft(x2, len, 1);
        for (int i = 0; i < len; i++) x1[i] = x1[i] * x2[i];
        fft(x1, len, -1);
        for (int i = 0; i < len; i++) {
            sum[i] = int(x1[i].x + 0.5);
        }
        while (sum[len] == 0 && len > 0) len--;
        for (int i = 0; i <= len; i++) printf("%d", sum[i]);
        puts("");
    }
}
```