---
title: 2019ICPC南京网络赛C Tsy's number 5
date: 2019-09-02 16:21:01
categories:
- ACM
- 题目
tags:
- 数学
- NTT
---

# 题意

计算 $$\sum^n_{i=1} \sum^n_{j=1} \phi(i) \phi(j) 2^{\phi(i) \phi(j)}$$。 结果模 $998244353$。

$1 \leq n \leq 1e5$

# 分析
设 $f_i = \sum^n_{k=1}[\phi(k) = i]$，即为 $1$ 到 $n$ 中 $\phi$ 为 $i$ 的个数。

$$
\begin{aligned}
\sum^n_{i=1} \sum^n_{j=1} \phi(i) \phi(j) 2^{\phi(i) \phi(j)} &= \sum^n_{i=1} \sum^n_{j=1}ijf_if_j2^{ij}  \\\\
&= 2\sum^n_{i=1} \sum^i_{j=1}ijf_if_j2^{ij} - \sum^n_{i=1}i^2f_i^22^{i^2} \\\\
&= 2\sum^n_{i=1} if_i\sum^i_{j=1}jf_j (\sqrt2)^{i^2 + j^2 - (i - j)^2} - \sum^n_{i=1}i^2f_i^22^{i^2} \\\\
&= 2\sum^n_{i=1} if_i(\sqrt2)^{i^2}\sum^i_{j=1}jf_j (\sqrt2)^{j^2}(\sqrt2)^{ - (i - j)^2} - \sum^n_{i=1}i^2f_i^22^{i^2} \\\\
\end{aligned}
$$

然后就可以直接 $NTT$ 了。

# 代码
```cpp
#include <bits/stdc++.h>
using namespace std;
const int mod = 998244353;
using ll = long long;
const int maxn = 1e6 + 5;
bool vis[maxn];
int prime[maxn], phi[maxn];
int f[maxn];
void CalPhi(int n) {
  memset(vis, 0, sizeof(vis));
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

  memset(f, 0, sizeof(f));
  for (int i = 1; i <= n; i++) f[phi[i]]++;
}
ll Pow(ll a, ll b) {
  ll ret = 1;
  while (b) {
    if (b & 1) ret = ret * a % mod;
    a = a * a % mod;
    b >>= 1;
  }
  return ret;
}
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
// /*
int A[maxn], B[maxn];

void multiply(int x[], int y[], int n, int m) {
  // n,m 表示长度 （阶数要加一）
  int len = 1;
  while (len <= (m + n)) len <<= 1;
  for (int i = 0; i < len; i++) A[i] = B[i] = 0;
  for (int i = 0; i <= n; i++) A[i] = x[i];
  for (int i = 0; i <= m; i++) B[i] = y[i];
  ntt(A, len, 1);
  ntt(B, len, 1);
  for (int i = 0; i < len; i++) x[i] = 1LL * A[i] * B[i] % mod;
  ntt(x, len, -1);
}
// */
}  // namespace ntt

const int TWO = 116195171;
int x[maxn], y[maxn];
ll work(int n) {
  for (ll i = 0; i < n; i++)
    x[i] = (i + 1) * f[i + 1] % mod * Pow(TWO, (i + 1) * (i + 1)) % mod;
  for (ll i = 0; i < n; i++) y[i] = Pow(Pow(TWO, i * i), mod - 2);
  ntt::multiply(x, y, n - 1, n - 1);
  ll ret = 0;
  for (ll i = 1; i <= n; i++)
    ret = ret + i * f[i] % mod * Pow(TWO, i * i) % mod * x[i - 1] % mod;
  return ret;
}

int main() {
  ntt::getwn();
  int T;
  scanf("%d", &T);
  while (T--) {
    int n;
    scanf("%d", &n);
    CalPhi(n);
    ll ans = 2LL * work(n) % mod;
    for (ll i = 1; i <= n; i++)
      ans = (ans - i * i % mod * f[i] % mod * f[i] % mod * Pow(2, i * i) % mod +
             mod) %
            mod;
    printf("%lld\n", ans);
  }
}
```
