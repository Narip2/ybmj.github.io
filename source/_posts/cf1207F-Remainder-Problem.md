---
title: cf1207F Remainder Problem
date: 2019-08-25 12:15:24
categories:
- ACM
- 题目
tags:
- 分块
---

[题目链接](https://codeforces.com/contest/1207/problem/F)

# 题意

$n$ 个数，初始值都为 $0$，有 $q$ 个操作：

1. $(1,x,y)$:  $a_x = a_x +  y$
2. $(2,x,y)$:  计算 $\sum_{i \in R(x,y)}a_i$， $R(x,y)$ 表示模 $x$ 余 $y$ 的数的集合

$1 \leq q \leq 500000, 1 \leq n \leq 500000$

# 分析

> 并不知道有什么数据结构可以直接维护模 $x$ 余 $y$ 的数的集合。

模 $x$ 余 $y$ 这个条件，对于 $x$ 较大的时候，符合条件的数字很少。

所以我们可以按根号分块，对于 $x$ 较小的那部分，我们直接维护 $b[x][y]$ 表示模 $x$ 余 $y$ 的答案，这样更新是 $sqrt(n)$ 的；对于 $x$ 较大的那部分，我们直接维护 $a[i]$ 表示这个数字的值，然后每次 $sqrt(n)$ 的查询答案。

# 代码

```cpp
#include <bits/stdc++.h>
using namespace std;
using ll = long long;
const int maxn = 5e5 + 5;
const int N = 1e3 + 5;
const int gap = sqrt(maxn);
int a[maxn];
int b[N][N];

int main() {
  // memset(b, 0, sizeof(b));
  int q;
  scanf("%d", &q);
  for (int _ = 0, op, x, y; _ < q; _++) {
    scanf("%d%d%d", &op, &x, &y);
    if (op == 1) {
      a[x] += y;
      for (int i = 1; i < gap; i++) {
        b[i][x % i] += y;
      }
    } else {
      ll ans = 0;
      if (x >= gap) {
        for (int i = 0; y + i * x < maxn; i++) {
          ans += a[y + i * x];
        }
      } else {
        ans = b[x][y];
      }
      printf("%lld\n", ans);
    }
  }
}
```
