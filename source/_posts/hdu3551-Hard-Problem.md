---
title: hdu3551 Hard Problem
date: 2019-10-30 16:05:59
categories:
- ACM
- 题目
tags:
- 一般图最大匹配
---

[题目链接](http://acm.hdu.edu.cn/showproblem.php?pid=3551)


## 题意

给一张 $n$ 个点 $m$ 条边的无向图。再给一个度数序列 $d_1, d_2 ... d_n$，问是否可以找到一个子图，使得子图中每个点的度数和对应的 $d_i$ 相同。

$1 \leq n \leq 50, 1 \leq m \leq 200$

## 分析

一个显然的想法是对度数拆点。

不妨假设一次匹配的意义为删边操作所带来度数减少。

那么设 $D_i$ 为 $i$ 点在原图中的度数，那么 $deg_i = D_i - d_i$ 就是将要减少的度数。

将每个点拆成 $deg_i$ 个点。 每条边拆成两个点。

跑一次最大匹配，如果是完美匹配，则证明可以找到这样一个子图。并且如果已匹配边的两个端点是原图中一条边拆成的两个点，那么表示这条边在原图中不会被删去。

匹配的顺序是优先去匹配点再去匹配边拆成的点，因为要保证每个点的度数都满足要求。


## 代码
```cpp
#include <bits/stdc++.h>
using namespace std;
const int maxn = 805;
using pii = pair<int, int>;
vector<pii> edges;
int deg[maxn];

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

int pre[maxn];
int main() {
  int n, m, T;
  int kase = 1;
  scanf("%d", &T);
  while (T--) {
    scanf("%d%d", &n, &m);
    for (int i = 1; i <= n; i++) deg[i] = 0;
    edges.clear();
    for (int i = 0, u, v; i < m; i++) {
      scanf("%d%d", &u, &v);
      deg[u]++, deg[v]++;
      edges.push_back({u, v});
    }
    pre[0] = 1;
    for (int i = 1, x; i <= n; i++) {
      scanf("%d", &x);
      deg[i] -= x;
      pre[i] = pre[i - 1] + deg[i];
    }
    int tot = pre[n] + m * 2 - 1;

    printf("Case %d: ", kase++);
    bool ok = true;
    for (int i = 1; i <= n; i++) {
      if (deg[i] < 0) {
        puts("NO");
        ok = false;
        break;
      }
    }
    if (!ok) continue;

    match.init(tot);
    for (int i = 0; i < m; i++) {
      int u = pre[n] + i * 2, v = u + 1;
      int x = edges[i].first, y = edges[i].second;
      for (int j = pre[x - 1]; j < pre[x]; j++) match.addEdge(j, u);
      for (int j = pre[y - 1]; j < pre[y]; j++) match.addEdge(j, v);
      match.addEdge(u, v);
    }
    int ans = match.work();
    for (int i = 1; i <= tot; i++) {
      if (match.match[i] == 0) {
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