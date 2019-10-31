---
title: zoj3316 Game
date: 2019-10-30 15:53:55
categories:
- ACM
- 题目
tags:
- 一般图最大匹配
---


[题目链接](https://zoj.pintia.cn/problem-sets/91827364500/problems/91827368225)

## 题意

一个 $19 \times 19$ 的棋盘，上面有 $n$ 个棋子。两个人交替每次拿走一个棋子，要求两次拿走的棋子的曼哈顿距离不超过 $L$。最后谁不能拿谁输。先手不受限制。 问最后谁输谁赢。

## 分析

一回合的操作可以看作一个匹配。

如果两颗棋子的曼哈顿距离小于 $L$，则建一条边。

如果图中存在完美匹配，那么先手必输，因为后手只要拿对应的匹配即可。

否则的话先手只要走一个未盖点，这样的话后手一定会走在匹配点上。接下来先手只要走对应的配偶，则必胜。因为从未盖点出发不会走到另外一个未盖点（否则就可以继续增广，就不是最大匹配了）。

## 代码
```cpp
#include <bits/stdc++.h>
using namespace std;
const int maxn = 400;
int x[maxn], y[maxn];

struct MaxMatch {
  vector<int> G[maxn];
  queue<int> Q;
  int n, clk;
  int match[maxn], par[maxn];
  int Type[maxn], pre[maxn], vis[maxn];

  void init(int n) {
    this->n = n;
    clk = 0;
    for (int i = 1; i <= n; i++) G[i].clear(), vis[i] = 0, match[i] = 0;
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
    for (int i = 1; i <= n; i++)
      if (!match[i]) ans += Augument(i);
    return ans;
  }
} match;

int n, L;
bool ok(int i, int j) {
  int ans = abs(x[i] - x[j]) + abs(y[i] - y[j]);
  return ans <= L;
}
int main() {
  while (~scanf("%d", &n)) {
    for (int i = 1; i <= n; i++) scanf("%d%d", x + i, y + i);
    scanf("%d", &L);
    match.init(n);
    for (int i = 1; i <= n; i++) {
      for (int j = i + 1; j <= n; j++) {
        if (ok(i, j)) match.addEdge(i, j);
      }
    }
    match.work();
    bool ok = true;

    for (int i = 1; i <= n; i++) {
      if (!match.match[i]) {
        ok = false;
        break;
      }
    }
    if (ok)
      puts("YES");
    else
      puts("NO");
  }
}
```