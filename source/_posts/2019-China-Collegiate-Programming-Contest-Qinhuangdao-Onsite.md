---
title: 2019 China Collegiate Programming Contest Qinhuangdao Onsite
date: 2019-10-15 15:50:58
categories:
- ACM
- 题目
tags:
- CCPC
---

[比赛链接](https://codeforces.com/gym/102361)

# G. Game on Chessboard

## 题意
给定一个 $n\times n$ 的棋盘，每个格子上有三种取值 `W`，`B`，`.`。每个格子都有一个对应权值 $a_{ij}$。且 `.` 的权值一定为 $0$。从格子上拿走对应元素需要付出对应权值的代价。问将所有格子都拿走需要付出的最小代价。

有一种特殊的操作是：可以同时拿一个`W`，和一个`B`。前提是这两个棋子左下部分都不存在棋子。这样做的花费是其权值差的绝对值。

$1 \leq n \leq 12$

## 分析

![](https://ybmj-blog-1256173108.cos.ap-shanghai.myqcloud.com/blog-picture/2019CCPC%E7%A7%A6%E7%9A%87%E5%B2%9BG.jpg)

观察发现，只有图中涂绿色的块可以消消乐。而只拿一个块的也话也只会拿绿色的块，因为只有拿了绿色的块，才会有更多的块可以消消乐。因此决策点就是是否拿绿色的块（角块）。

状态表示可以用红色的轮廓线来表示，比如竖直的是 $0$，水平的是 $1$。图中的状态就可以表示为 $01001011$。

每次的决策就是枚举消消乐，再枚举单独拿一个块的转移。

最后快乐 $dp$ 即可。

## 代码

```cpp
#include <bits/stdc++.h>
using namespace std;
const int maxn = (1 << 24) + 5;
char maze[15][15];
int a[15][15];
int dp[maxn];

int n;
struct Node {
  int r, c, pos;
};

int dfs(int stat) {
  if (dp[stat] != -1) return dp[stat];
  int r = 0, c = 0, szb = 0, szw = 0;
  int &ret = dp[stat];
  ret = 1e9;
  Node B[25], W[25];

  for (int i = 2 * n - 1; i > 0; i--) {
    if (!(stat >> i & 1) && (stat >> (i - 1) & 1)) {
      int nxt = stat ^ (3 << (i - 1));
      ret = min(ret, dfs(nxt) + a[r][c]);
      if (maze[r][c] == 'B') B[szb++] = {r, c, i};
      if (maze[r][c] == 'W') W[szw++] = {r, c, i};
    }
    if (stat >> i & 1)
      c++;
    else
      r++;
  }

  for (int i = 0; i < szb; i++) {
    int ip = B[i].pos, ir = B[i].r, ic = B[i].c;
    int tmp = stat ^ (3 << (ip - 1));
    for (int j = 0; j < szw; j++) {
      int jp = W[j].pos, jr = W[j].r, jc = W[j].c;
      int nxt = tmp ^ (3 << (jp - 1));
      ret = min(ret, dfs(nxt) + abs(a[jr][jc] - a[ir][ic]));
    }
  }

  return ret;
}
int main() {
  scanf("%d", &n);
  for (int i = 0; i < n; i++) scanf("%s", maze[i]);
  for (int i = 0; i < n; i++)
    for (int j = 0; j < n; j++) scanf("%d", &a[i][j]);
  memset(dp, -1, sizeof(dp));
  int s = (1 << n) - 1;
  int t = s << n;
  dp[t] = 0;
  int ans = dfs(s);
  printf("%d\n", ans);
}
```
