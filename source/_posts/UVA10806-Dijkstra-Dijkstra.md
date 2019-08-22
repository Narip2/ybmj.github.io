---
title: 'UVA10806 Dijkstra, Dijkstra.'
date: 2019-08-22 18:16:16
categories:
- ACM
- 题目
tags:
- 最短路
---

[题目链接](https://uva.onlinejudge.org/index.php?option=com_onlinejudge&Itemid=8&page=show_problem&problem=1747)

# 题意
求一条 $s$ 到 $t$ 的最短往返路，要求不能走重复的边。

# 分析

考虑网络流中的退流操作。

先跑一遍s到t的最短路，然后把这条路的权值都更新为正无穷，保证之后不会再走第二遍。然后把相应地反向边的权值变成相反数。

再跑一次s到t的最短路，此时如果走到之前的反向边，则相当于“退流”。

# 代码
```cpp
#include <bits/stdc++.h>
using namespace std;
const int maxn = 105;
const int INF = 0x3f3f3f3f;
typedef pair<int, int> pii;
int G[maxn][maxn];
int p[maxn], d[maxn];
int n, m;

void dijsktra(int s, int t) {
  for (int i = 1; i <= n; i++) d[i] = INF;
  d[s] = 0;
  priority_queue<pii, vector<pii>, greater<pii> > pq;
  pq.push(make_pair(0, s));
  while (!pq.empty()) {
    pii now = pq.top();
    pq.pop();
    int v = now.second;
    if (d[v] < now.first) continue;
    for (int i = 1; i <= n; i++) {
      if (G[v][i] < INF) {
        if (d[i] > d[v] + G[v][i]) {
          d[i] = d[v] + G[v][i];
          p[i] = v;
          pq.push({d[i], i});
        }
      }
    }
  }
}

int main() {
  while (~scanf("%d", &n)) {
    if (!n) break;
    memset(G, 0x3f, sizeof(G));
    scanf("%d", &m);
    for (int i = 0, u, v, t; i < m; i++) {
      scanf("%d%d%d", &u, &v, &t);
      G[u][v] = t;
      G[v][u] = t;
    }
    dijsktra(1, n);
    int ans = d[n];
    p[1] = -1;
    for (int u = n; u != 1; u = p[u]) {
      G[p[u]][u] = INF;
      G[u][p[u]] *= -1;
    }
    dijsktra(1, n);
    ans += d[n];
    if (ans >= INF)
      puts("Back to jail");
    else
      printf("%d\n", ans);
  }
}
/*
2
1
1 2 999
3
3
1 3 10
2 1 20
3 2 50
9
12
1 2 10
1 3 10
1 4 10
2 5 10
3 5 10
4 5 10
5 7 10
6 7 10
7 8 10
6 9 10
7 9 10
8 9 10
0

*/
```
