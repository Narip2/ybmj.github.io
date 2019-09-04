---
title: 2019ICPC南京网络赛I Washing clothes
date: 2019-09-03 13:59:06
categories:
- ACM
- 题目
tags:
- 暴力
---

# 题意
有 $n$ 个人，每个人都带着一件要洗的衣服在 $t_i$ 的时刻到达洗衣房。洗衣房内只有一个洗衣机，一次只能洗一件衣服。每个人都可以选择机洗或手洗，机洗所用时间为 $x$，手洗所用时间为 $y$。请输出当 $x \in [1,y]$ 时，所对应洗完所有衣服的最短时间。

$1 \leq N,y \leq 10^6$

$0 \leq t_i \leq 10^9$

# 分析

对于每个确定的 $x$，一定存在一个位置 $p$，使得小于 $p$ 位置的都手洗，大于等于 $p$ 位置的都机洗。

用数学语言表示：

$f(i) = t_i + y$：表示第 $i$ 个人手洗结束的时间

$g(i) = t_i + (n - i + 1) \times x$：表示从第 $i$ 个人开始机洗所用的最短时间。

$h(i) = max(h(i+1), g(i))$：表示 $g$ 的后缀最大值。

那么答案为 $ans = min\{max(f(i-1), h(i))\}$。

观察发现： $f$ 是单调递增的，$h$ 是单调递减的，那么 $ans$ 实际上就是下凸壳的最小值。并且随着 $x$ 的增大，最值的位置逐渐右移。

<!-- 对于每个 $x$ 考虑最后一个位置 $p$ 使得 -->

考虑位置 $p$：

如果有 $t_p + (n - p + 1) \times x > t_p + y$，那么极值点一定在 $p$ 的左边。

那么我们可以算出满足 $t_p + (n - p + 1) \times x \leq t_p + y$ 的位置 $p$。

$p = n + 1 - \frac{y}{x}$

那么极值点一定是在 $p$ 之后取。我们发现 $\frac{y}{x}$ 这部分实际上是个调和级数，因此可以暴力计算。总的复杂度为 $n\log(n)$。

# 代码
```cpp
#include <bits/stdc++.h>
using namespace std;
const int maxn = 1e6 + 6;
using ll = long long;
const ll INF = 1e18;
int t[maxn];
ll h[maxn];

int main() {
  int n, y;
  while (~scanf("%d%d", &n, &y)) {
    for (int i = 1; i <= n; i++) scanf("%d", t + i);
    sort(t + 1, t + 1 + n);
    t[0] = -y;
    h[n + 1] = -INF;
    for (int x = 1; x <= y; x++) {
      int p = max(1, n + 1 - y / x);
      ll ans = INF;
      for (int j = n; j >= p; j--)
        h[j] = max(h[j + 1], t[j] + 1LL * (n - j + 1) * x);
      for (int j = n; j >= p; j--) ans = min(ans, max(h[j], (ll)t[j - 1] + y));
      printf("%lld%c", ans, x == y ? '\n' : ' ');
    }
  }
}

/*
y < (n - p + 1) * x;

p < n + 1 - y / x
*/
```
