---
title: cf102059G Fascination Street
date: 2019-08-26 20:54:45
categories:
- ACM
- 题目
tags:
- 动态规划
---

# 题意

路上有 $n$ 盏灯，点亮一盏灯的花费是 $w_i$。你可以交换 $k$ 次。要求最小化花费，使得每个位置左右或自己至少有一个位置的灯点亮。

# 分析

首先，**一定是交换一个点亮的灯和不点亮的灯**。其次，**每个位置是否点灯取决于之前两个位置的状态**。

令 $dp[i][S][a][b]$ 表示考虑到第 $i$ 个位置，上两个位置的情况为 $S$ ，换出 $a$ 个点亮的灯，换入 $b$ 个不点亮的灯的最小代价。

转移一共有四种，点亮、不点亮、点亮后换出、不点亮并换来一个点亮的。

需要注意初始化的边界，复杂度 $O(nk^2)$

# 代码
```cpp
#include <bits/stdc++.h>
using namespace std;
using ll = long long;
const int maxn = 3e5 + 5;
ll dp[maxn][2][2][10][10];
int w[maxn];
int main() {
  int n, k;
  scanf("%d%d", &n, &k);
  for (int i = 1; i <= n; i++) scanf("%d", w + i);
  memset(dp, 0x3f, sizeof(dp));
  dp[1][1][0][0][0] = 0;
  for (int i = 1; i <= n; i++) {
    for (int a = 0; a < 2; a++) {
      for (int b = 0; b < 2; b++) {
        for (int x = 0; x <= k; x++) {
          for (int y = 0; y <= k; y++) {
            ll &v = dp[i][a][b][x][y];
            dp[i + 1][b][1][x][y] = min(dp[i + 1][b][1][x][y], v + w[i]);
            if (a + b > 0)
              dp[i + 1][b][0][x][y] = min(dp[i + 1][b][0][x][y], v);
            if (a + b > 0 && x + 1 <= k)
              dp[i + 1][b][0][x + 1][y] =
                  min(dp[i + 1][b][0][x + 1][y], v + w[i]);
            if (y + 1 <= k)
              dp[i + 1][b][1][x][y + 1] = min(dp[i + 1][b][1][x][y + 1], v);
          }
        }
      }
    }
  }
  ll ans = 1e18;
  for (int a = 0; a < 2; a++)
    for (int b = 0; b < 2; b++)
      for (int x = 0; x <= k; x++)
        if (a + b > 0) ans = min(ans, dp[n + 1][a][b][x][x]);
 
  printf("%lld\n", ans);
}
```