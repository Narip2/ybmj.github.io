---
title: hdu3446 daizhenyang's chess
date: 2019-10-30 16:21:12
categories:
- ACM
- 题目
tags:
- 一般图最大匹配
---

[题目链接](http://acm.hdu.edu.cn/showproblem.php?pid=3446)

## 题意
给定一个 $n \times m$ 大小的棋盘，棋盘上有一些空位置。给定棋子的初始位置，两人轮流移动棋子，不能走的人输。 

规定：
1. 走过的位置不能再走。
2. 每次所走的位置相对于原来的位置的偏移是给定的。比如给了两种走法 $(+1,+2), (-1,0)$。

## 分析

将图中每个空位置看作点，“相邻”的点建一条边。跑一次最大匹配。

然后再将起始点加入图中，跑一次最大匹配。

如果两次的匹配数相等，则说明从起始点这个未盖点出发，没有找到增广路。即不管先手怎么走，后手只要走其对应的配偶就必胜。

如果匹配数增加，则说明有一条从起始点出发的增广路。即交替路的长度为奇数，先手必胜。


## 代码
```cpp
#include <bits/stdc++.h>
using namespace std;
const int maxn = 300;
char maze[20][20];
int dr[] = {-2, -2, -2, -2, -1, -1, -1, -1, -1, 0,
            0,  1,  1,  1,  1,  1,  2,  2,  2,  2};
int dc[] = {-2, -1, 1,  2, -2, -1, 0,  1,  2, -1,
            1,  -2, -1, 0, 1,  2,  -2, -1, 1, 2};
int n, m;

struct MaxMatch {
  vector<int> G[maxn];
  queue<int> Q;
  int n, clk;
  int match[maxn];
  int par[maxn];
  int Type[maxn];
  int pre[maxn];
  int vis[maxn];

  void init(int n) {
    this->n = n;
    for (int i = 1; i <= n; i++) G[i].clear();
  }
  void addEdge(int u, int v) {
    G[u].push_back(v);
    G[v].push_back(u);
  }

  int LCA(int x, int y) {
    clk++;
    x = par[x], y = par[y];
    while (vis[x] != clk) {
      if (x) {
        vis[x] = clk;
        x = par[pre[match[x]]];
      }
      swap(x, y);
    }
    return x;
  }
  void blossom(int x, int y, int lca) {
    while (par[x] != lca) {
      pre[x] = y;
      y = match[x];
      if (Type[y] == 1) {
        Type[y] = 0;
        Q.push(y);
      }
      par[x] = par[y] = par[lca];
      x = pre[y];
    }
  }
  int Augument(int s) {
    for (int i = 1; i <= n; i++) par[i] = i, Type[i] = -1;
    Q = queue<int>();
    Type[s] = 0;
    Q.push(s);
    while (!Q.empty()) {
      int u = Q.front();
      Q.pop();
      for (auto &v : G[u]) {
        if (Type[v] == -1) {
          pre[v] = u;
          Type[v] = 1;
          if (!match[v]) {
            for (int to = v, from = u; to; from = pre[to]) {
              match[to] = from;
              swap(match[from], to);
            }
            return true;
          }
          Type[match[v]] = 0;
          Q.push(match[v]);
        } else if (Type[v] == 0 && par[u] != par[v]) {
          int lca = LCA(u, v);
          blossom(u, v, lca);
          blossom(v, u, lca);
        }
      }
    }
    return false;
  }

  int work() {
    int ans = 0;
    clk = 0;
    for (int i = 1; i <= n; i++) vis[i] = 0, match[i] = 0;
    for (int i = 1; i <= n; i++)
      if (!match[i]) ans += Augument(i);
    return ans;
  }
} match;

int id[20][20];

int build() {
  int tot = 0;
  for (int i = 0; i < n; i++) {
    for (int j = 0; j < m; j++) {
      id[i][j] = 0;
      if (maze[i][j] == '.') id[i][j] = ++tot;
    }
  }

  for (int i = 0; i < n; i++) {
    for (int j = 0; j < m; j++) {
      if (!id[i][j]) continue;
      for (int k = 0; k < 20; k++) {
        int nr = i + dr[k];
        int nc = j + dc[k];
        if (nr < 0 || nr >= n || nc < 0 || nc >= m || !id[nr][nc]) continue;
        int x = id[i][j], y = id[nr][nc];
        match.addEdge(x, y);
      }
    }
  }
  match.n = tot++;
  match.work();
  for (int i = 0; i < n; i++) {
    for (int j = 0; j < m; j++) {
      if (maze[i][j] == 'K') {
        for (int k = 0; k < 20; k++) {
          int nr = i + dr[k];
          int nc = j + dc[k];
          if (nr < 0 || nr >= n || nc < 0 || nc >= m || !id[nr][nc]) continue;
          match.addEdge(tot, id[nr][nc]);
        }
        break;
      }
    }
  }
  match.n = tot;
  match.vis[tot] = match.match[tot] = 0;
  int res = match.Augument(tot);

  match.init(tot);
  return res == 0;
}

int main() {
  int T, kase = 1;
  scanf("%d", &T);
  while (T--) {
    scanf("%d%d", &n, &m);
    for (int i = 0; i < n; i++) scanf("%s", maze[i]);
    printf("Case #%d: daizhenyang ", kase++);
    if (build())
      puts("lose");
    else
      puts("win");
  }
}
```