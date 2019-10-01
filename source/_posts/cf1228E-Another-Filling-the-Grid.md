---
title: cf1228E Another Filling the Grid
date: 2019-10-01 19:43:24
categories:
- ACM
- 题目
tags:
- 动态规划
---

[题目链接](https://codeforces.com/contest/1228/problem/E)

# 题目
给定一个 $n \times n$ 的矩阵，每个格子可以填一个 $[1,k]$ 的数字。要求每行最小的数字和每列最小的数字都是 $1$ 。 问满足条件的填法一共有几种？

$1 \leq n \leq 250, 1 \leq k \leq 19^9$

# 分析

不难发现每一行和每一列都是等价的，因此我们 $dp$ 的状态表示应该只与已经填好的行数和列数有关。

设 $dp[i][j]$ 表示 $i$ 行中，有 $j$ 行不满足条件， $n-j$ 行满足条件。使得这个状态满足条件的填法。

$dp[i][0] = (k^n - (k-1)^n)^i$ 所有列都满足条件，每一行至少有一个 $1$。

$dp[1][j] = k^{n-j}$  有 $j$ 行不满足条件，那么这 $j$ 行就要填 $1$，其他随便填。

$dp[i][j] = dp[i-1][j] \times (k-1)^{j} \times (k^{n-j} - (k-1)^{n-j}) + \sum_{t=1}^{j} \tbinom{j}{t} \times (k-1)^{j-t} \times k^{n-j} \times dp[i-1][j-t]$

# 代码

```cpp
#include <bits/stdc++.h>
using namespace std;
using ll = long long;
const int mod = 1e9 + 7;
const int maxn = 255;
int C[maxn][maxn];
int powk[maxn], powkMinusOne[maxn];
int n, k;
 
ll Pow(ll a, ll b) {
  ll ret = 1;
  while (b) {
    if (b & 1) ret = ret * a % mod;
    a = a * a % mod;
    b >>= 1;
  }
  return ret;
}
int dp[maxn][maxn];
 
int main() {
  C[0][0] = 1;
  for (int i = 1; i < maxn; i++) {
    C[i][0] = 1;
    for (int j = 1; j <= i; j++) {
      C[i][j] = (C[i - 1][j - 1] + C[i - 1][j]) % mod;
    }
  }
  cin >> n >> k;
 
  powk[0] = powkMinusOne[0] = 1;
  for (int i = 1; i <= n; i++) {
    powk[i] = 1LL * powk[i - 1] * k % mod;
    powkMinusOne[i] = 1LL * powkMinusOne[i - 1] * (k - 1) % mod;
  }
 
  int var1 = (Pow(k, n) - Pow(k - 1, n) + mod) % mod;
  for (int i = 1; i <= n; i++) {
    dp[i][0] = Pow(var1, i);
    dp[1][i] = powk[n - i];
  }
  for (int i = 2; i <= n; i++) {
    for (int j = 1; j <= n; j++) {
      int var2 = (1LL * powk[n - j] - powkMinusOne[n - j] + mod) % mod;
      dp[i][j] = 1LL * powkMinusOne[j] * var2 % mod * dp[i - 1][j] % mod;
      for (int t = 1; t <= j; t++)
        dp[i][j] += 1LL * C[j][t] * powkMinusOne[j - t] % mod * powk[n - j] %
                    mod * dp[i - 1][j - t] % mod,
            dp[i][j] %= mod;
    }
  }
  cout << dp[n][n] << endl;
}
```