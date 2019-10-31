---
title: 2019 CCPC XiaMen Onsite
date: 2019-10-30 18:27:25
categories:
- ACM
- 题目
tags:
- 一般图最大匹配
---

# B-Zayin and Elements
## 题意
$n$ 个怪兽，$m$ 个骑士，每个骑士有三个属性 $a_i,b_i,c_i$。每个骑士可以消灭指定的一些怪兽。
当一个骑士消灭一个怪兽时，如果其 $a_i > 0$，那么会消耗 $1$ 单位的 $a_i$。否则如果其 $b_i > 0$，那么会消耗 $0.5$ 单位的 $b_i$。否则如果其 $c_i > 0$，那么会消耗 $1$ 单位的 $c_i$。问消灭完所有怪兽后，最大的 $\sum^m_{i=1} \lfloor b_i + c_i \rfloor$ 是多少。

$1 \leq n \leq 100, 1 \leq m \leq 20, 0 \leq a_i + b_i + c_i \leq 10$

## 分析
比赛的时候还不会一般图匹配，所以一直当网络流做。一直卡在没有办法解决 $b$ 的花费。因为用 $b$ 消灭一个怪兽的代价和消灭两个怪兽的代价是相同的。

对于每单位的 $a$ 建一个点 $a_1$，对每单位的 $b$ 建两个点 $b_1,b_2$，对每单位的 $c$ 建两个点 $c_1,c_2$。

对于某个怪兽 $x$，建 $(x,a_1), (x,b_1), (x,b_2), (b_1,b_2), (x,c_1), (c_1,c_2)$。

然后跑一般图最大匹配。注意要优先匹配怪兽，因为首先要满足的条件是所有怪兽被消灭。然后再去顺序匹配 $a,b,c$。因为带花树在匹配数不增加的情况下不会改变已匹配的边，所以匹配要有一个优先级。

有解当且仅当所有的怪兽都是匹配点。

答案就是最大匹配数减去怪兽的数量。

## 代码
```cpp
#include <bits/stdc++.h>
using namespace std;

const int maxn = 1000;
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

} match;

int vec[1000];
int main() {
  int T;
  scanf("%d", &T);
  while (T--) {
    int n, m;
    scanf("%d%d", &n, &m);
    int tot = n;
    vector<int> av, bv, cv;
    for (int i = 1, cnt, a, b, c; i <= m; i++) {
      scanf("%d%d%d%d", &a, &b, &c, &cnt);
      for (int j = 0; j < cnt; j++) scanf("%d", vec + j);

      for (int j = 0; j < a; j++) {
        int x = ++tot;
        av.push_back(x);
        for (int k = 0; k < cnt; k++) match.addEdge(x, vec[k]);
      }
      for (int j = 0; j < 2 * b; j += 2) {
        int x = ++tot;
        int y = ++tot;
        bv.push_back(x);
        bv.push_back(y);
        match.addEdge(x, y);
        for (int k = 0; k < cnt; k++) {
          match.addEdge(x, vec[k]);
          match.addEdge(y, vec[k]);
        }
      }
      for (int j = 0; j < 2 * c; j += 2) {
        int x = ++tot;
        int y = ++tot;
        cv.push_back(x);
        cv.push_back(y);
        match.addEdge(x, y);
        for (int k = 0; k < cnt; k++) match.addEdge(x, vec[k]);
      }
    }

    int ans = 0;
    bool ok = true;
    match.n = tot;
    for (int i = 1; i <= n; i++) {
      if (!match.match[i]) {
        int x = match.Augument(i);
        ans += x;
        if (!x) {
          ok = false;
          break;
        }
      }
    }
    if (!ok)
      puts("-1");
    else {
      for (auto &v : av)
        if (!match.match[v]) ans += match.Augument(v);
      for (auto &v : bv)
        if (!match.match[v]) ans += match.Augument(v);
      for (auto &v : cv)
        if (!match.match[v]) ans += match.Augument(v);
      printf("%d\n", ans - n);
    }

    match.init(tot);
  }
}
```
