---
title: FWT 快速沃尔什变换
date: 2019-11-06 22:35:13
categories:
- ACM
- 数学
tags:
- FWT
---

# FWT 快速沃尔什变换

目前我了解的FWT的功能就是在 $O(n\log n) 时间复杂度内求：$

$C_k = \sum_{i|j=k}a_ib_j$

$C_k = \sum_{i\&j=k}a_ib_j$

$C_k = \sum_{i\oplus j=k}a_ib_j$

---


其思想和 $FFT$ 很类似。把原多项式 $A$ 转化成另一个多项式 $FWT(A)$。

然后 $A \oplus B = IFWT(FWT(A) \times FWT(B))$

其中 $\oplus$ 表示一些位运算。

然后接下来的问题是针对不同的位运算，去找出如何求 $FWT$ 和 $IFWT$。


## 一些定义

$A = [a_0, a_1, a_2, ..., a_{n-1}]$

设 $A$ 的长度为 $2$ 的幂次，将 $A = [A_0, A_1]$ 分成长度相同的两部分。

## OR

$C = A|B = [\sum_{i|j=0}a_ib_j, \sum_{i|j=1}a_ib_j, \sum_{i|j=2}a_ib_j, ..., \sum_{i|j=k}a_ib_j]$

$C_k = \sum_{i|j=k}a_ib_j$


$FWT(A) = [\sum_{i | 0 = 0}a_i, \sum_{i | 1 = 1}a_i, \sum_{i | 2 = 2}a_i, ..., \sum_{i | (n-1) = (n-1)}a_{n-1}]$


### FWT 变换所满足的规律

$$FWT(A) = [FWT(A_0), FWT(A_0 + A_1)]$$

### FWT 变换所满足的性质

$FWT(A+B) = FWT(A) + FWT(B)$

$FWT(A|B) = FWT(A) \times FWT(B)$

$FWT(A|B) = FWT([(A|B)_0, (A|B)_1])$
$ = [FWT((A|B)_0), FWT((A|B)_0 + (A|B)_1)]$
$ = [FWT(A_0 | B_0), FWT(A_0 | B_0 + A_0 | B_1 + A_1 | B_0 + A_1 | B_1)]$
$ = [FWT(A_0) \times FWT(B_0), FWT(A_0) \times FWT(B_0) + FWT(A_0) \times FWT(B_1) + FWT(A_1) \times FWT(B_0) + FWT(A_1) \times FWT(B_1))]$
$ = [FWT(A_0) \times FWT(B_0), (FWT(A_0) + FWT(A_1)) \times (FWT(B_0) + FWT(B_1))]$
$ = [FWT(A_0), FWT(A_0 + A_1)] \times [FWT(B_0), FWT(B_0 + B_1)]$
$ = FWT(A) \times FWT(B)$

### IFWT 变换所满足的规律

$$IFWT(A) = [IFWT(A_0), IFWT(A_1) - IFWT(A_0)]$$

## AND

$C = A\&B = [\sum_{i\&j=0}a_ib_j, \sum_{i\&j=1}a_ib_j, \sum_{i\&j=2}a_ib_j, ..., \sum_{i\&j=k}a_ib_j]$

$C_k = \sum_{i\&j=k}a_ib_j$

### FWT 变换所满足的规律

$$FWT(A) = [FWT(A_0 + A_1), FWT(A_1)]$$

### FWT 变换所满足的性质

$FWT(A+B) = FWT(A) + FWT(B)$

$FWT(A\&B) = FWT(A) \times FWT(B)$

### IFWT 变换所满足的规律

$$IFWT(A) = [IFWT(A_0) - IFWT(A_1), IFWT(A_1)]$$

## XOR

$C = A\oplus B = [\sum_{i\oplus j=0}a_ib_j, \sum_{i\oplus j=1}a_ib_j, \sum_{i\oplus j=2}a_ib_j, ..., \sum_{i\oplus j=k}a_ib_j]$

$C_k = \sum_{i\oplus j=k}a_ib_j$

### FWT 变换所满足的规律

$$FWT(A) = [FWT(A_0) + FWT(A_1), FWT(A_0) - FWT(A_1)]$$

### FWT 变换所满足的性质

$FWT(A+B) = FWT(A) + FWT(B)$

$FWT(A\oplus B) = FWT(A) \times FWT(B)$

### IFWT 变换所满足的规律

$$IFWT(A) = [\frac{IFWT(A_0) + IFWT(A_1)}{2}, \frac{IFWT(A_0) - IFWT(A_1)}{2}$$


# 代码

[洛谷P4717](https://www.luogu.org/problem/P4717)

```cpp
#include <bits/stdc++.h>
using namespace std;
const int mod = 998244353;
const int maxn = 1e6 + 6;
int a[maxn], b[maxn];
int inv2 = (mod + 1) / 2;

// 空间要开到2的幂次（清空也要清空到2的幂次）
template <class T>
void fwt(int a[], int m, T f) {
  for (int i = 1; i < m; i <<= 1)
    for (int p = i << 1, j = 0; j < m; j += p)
      for (int k = 0; k < i; k++) f(a[j + k], a[i + j + k]);
}
inline void AND(int &a, int &b) { (a += b) %= mod; }
inline void OR(int &a, int &b) { (b += a) %= mod; }
inline void XOR(int &a, int &b) {
  int l = a, r = b;
  a = (l + r) % mod, b = (l - r + mod) % mod;
}
inline void rAND(int &a, int &b) { a = (a - b + mod) % mod; }
inline void rOR(int &a, int &b) { b = (b - a + mod) % mod; }
inline void rXOR(int &a, int &b) {
  int l = a, r = b;
  a = 1LL * (l + r) % mod * inv2 % mod;
  b = 1LL * (l - r + mod) % mod * inv2 % mod;
}

int c[maxn], d[maxn];
int n, N;

void solve(int a[], int b[], int op) {
  for (int i = 0; i < N; i++) c[i] = a[i], d[i] = b[i];
  if (op == 0) {
    fwt(c, N, OR);
    fwt(d, N, OR);
    for (int i = 0; i < N; i++) c[i] = 1LL * c[i] * d[i] % mod;
    fwt(c, N, rOR);
  } else if (op == 1) {
    fwt(c, N, AND);
    fwt(d, N, AND);
    for (int i = 0; i < N; i++) c[i] = 1LL * c[i] * d[i] % mod;
    fwt(c, N, rAND);
  } else {
    fwt(c, N, XOR);
    fwt(d, N, XOR);
    for (int i = 0; i < N; i++) c[i] = 1LL * c[i] * d[i] % mod;
    fwt(c, N, rXOR);
  }
  for (int i = 0; i < N; i++) printf("%d ", c[i]);
  puts("");
}
int main() {
  scanf("%d", &n);
  N = 1 << n;
  for (int i = 0; i < N; i++) scanf("%d", a + i);
  for (int i = 0; i < N; i++) scanf("%d", b + i);
  solve(a, b, 0);
  solve(a, b, 1);
  solve(a, b, 2);
}
```