---
title: uoj171 挑战NPC
date: 2019-10-30 15:40:15
categories:
- ACM
- 题目
tags:
- 一般图最大匹配
---

[题目链接](http://uoj.ac/problem/171)

## 题意
$n$ 个球，$m$ 个球筐，每个球筐最多装 $3$ 个球。每个球只能放进特定的球筐中。每个球都必须放进一个球筐中。
如果一个球筐内的球数小于等于 $1$，则称这个球筐是半空的。求半空的球筐最多有多少个？

$1 \leq n \leq 3m, 1 \leq m \leq 100$

## 分析

如果题目只要求可行性的话，只要拆点二分图匹配即可。

每个球筐拆成三个点 $a, b, c$。

但是因为要求半空的球筐数量最多，所以我们希望在满足每个球都能放进一个球筐中的前提下，优先去匹配球筐内的点。（球筐内部如果存在一个匹配，那么最多只有一个球可以和这个球筐进行匹配）。

因此图就不是二分图了，就要用到一般图匹配。

对于每个球筐，我们将其拆成三个点后，只要在任意两个点连一条即可（如 $b,c$ 之间）。因为这三个点本质上是等价的。

因为要先满足每个球都能放进一个球筐中，所以我们应该优先去匹配球，即从每个球开始找增广路。

最后有解的条件是每个球都匹配到了球筐。

答案就是最大匹配数减去球数。

## 代码
```cpp
#include <bits/stdc++.h>
using namespace std;
const int maxn = 605;

struct MaxMatch {
  vector<int> G[maxn];
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
  queue<int> Q;
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
    for (int i = 1; i <= n; i++) par[i] = i;
    memset(Type, -1, sizeof(Type));
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

int main() {
  int T;
  scanf("%d", &T);
  while (T--) {
    int n, m, e;
    scanf("%d%d%d", &n, &m, &e);
    match.init(n + 3 * m);
    for (int i = 0; i < m; i++) match.addEdge(n + i * 3 + 1, n + i * 3 + 2);
    for (int i = 0, u, v; i < e; i++) {
      scanf("%d%d", &u, &v);
      match.addEdge(u, n + (v - 1) * 3 + 1);
      match.addEdge(u, n + (v - 1) * 3 + 2);
      match.addEdge(u, n + (v - 1) * 3 + 3);
    }
    int ans = match.work();
    printf("%d\n", ans - n);
    for (int i = 1; i <= n; i++)
      printf("%d ", (match.match[i] - n - 1) / 3 + 1);
    puts("");
  }
}
```