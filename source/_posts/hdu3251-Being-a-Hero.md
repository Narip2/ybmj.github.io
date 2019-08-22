---
title: hdu3251 Being a Hero
date: 2019-08-20 16:50:37
categories:
- ACM
- 题目
tags:
- 网络流
---

[题目链接]()

# 题意
有 $n$ 个城市，编号为 $[1,n]$，有 $m$ 条有向边，每条边有个权值 $w_i$，给定一个大小为 $f$ 的点集 $V$，每个点都有一个权值 $c_i$，你可以从中选择一些点并获得它的权值。但是要求从 $1$ 号点出发，不能够到达任何你选的点。你可以在选点之前切掉一些边，切掉边的代价为边的权值。最后获得的价值为所选点的权值和减去切掉边的权值和。 求获得的最大价值，以及应该选择切掉哪些边？

$1 \leq n \leq 10^3, 1 \leq m \leq 10^5$

# 分析

类似最大权闭合子图的建图。

将所有边反向，然后建立源点 $s$，对于点集 $V$ 中的每个点 $u$，建立 $(s,u)$ 容量为权值的边。

我们先默认 $V$ 中所有的点都被选，然后考虑要去掉哪些点或去掉哪些边。 这实际上就成为了一个最小割的问题。

在上面的建图中跑最小割。 如果割的是从源点出发的边，则表示这个点不会被选。如果割的是其它边，则表示需要割掉这些边。

挺经典的建图，如果忘了可以去复习最大权闭合子图的建图方式。


# 代码

```cpp
#include <bits/stdc++.h>
using namespace std;

const int maxn = 2e5;
const int INF = 0x3f3f3f3f;
int edge_id[maxn];

struct Edge {
  int u, v, cap, flow;
  Edge(int u = 0, int v = 0, int c = 0, int f = 0)
      : u(u), v(v), cap(c), flow(f) {}
};
struct Dinic {
  int n, s, t;
  vector<Edge> edges;
  vector<int> G[maxn];
  int d[maxn], cur[maxn];
  void init(int n) {
    this->n = n;
    for (int i = 0; i <= n; i++) G[i].clear();
    edges.clear();
  }
  int addEdge(int u, int v, int c) {
    edges.push_back({u, v, c, 0});
    edges.push_back({v, u, 0, 0});
    int m = edges.size();
    G[u].push_back(m - 2);
    G[v].push_back(m - 1);
    return m - 2;
  }

  bool BFS() {
    for (int i = 0; i <= n; i++) d[i] = 0;
    d[s] = 1;
    queue<int> q;
    q.push(s);
    while (!q.empty()) {
      int u = q.front();
      q.pop();
      int v;
      for (auto &id : G[u]) {
        auto &e = edges[id];
        v = e.v;
        if (!d[v] && e.cap > e.flow) {
          d[v] = d[u] + 1;
          q.push(v);
        }
      }
    }
    if (d[t]) return true;
    return false;
  }
  int DFS(int u, int f) {
    if (u == t || f == 0) return f;
    int v, a, flow = 0;
    for (int &i = cur[u]; i < G[u].size(); i++) {
      auto &e = edges[G[u][i]];
      v = e.v;
      if (d[v] == d[u] + 1 && (a = DFS(v, min(f, e.cap - e.flow))) > 0) {
        e.flow += a;
        edges[G[u][i] ^ 1].flow -= a;
        flow += a;
        f -= a;
        if (f == 0) break;
      }
    }
    return flow;
  }
  int MaxFlow(int s, int t) {
    this->s = s, this->t = t;
    int flow = 0;
    while (BFS()) {
      for (int i = 0; i <= n; i++) cur[i] = 0;
      flow += DFS(s, INF);
    }
    return flow;
  }

  bool vis[maxn];

  int edge_id[maxn];  // 每条边的编号
  vector<int> cut;    // 割边的编号
  void findCut(int m) {
    cut.clear();
    for (int i = 0; i <= n; i++) vis[i] = 0;
    queue<int> q;
    q.push(s);
    vis[s] = 1;
    while (!q.empty()) {
      int u = q.front();
      q.pop();
      for (auto &id : G[u]) {
        auto &e = edges[id];
        if (!vis[e.v] && e.cap > e.flow) {
          vis[e.v] = 1;
          q.push(e.v);
        }
      }
    }
    for (int i = 0; i < m; i++) {
      auto &e = edges[edge_id[i]];
      if (vis[e.u] == vis[e.v]) continue;
      if (e.cap == e.flow) cut.push_back(i + 1);  // 1-index
    }
  }
} dinic;

int main() {
  int T, kase = 1;
  scanf("%d", &T);
  while (T--) {
    int n, m, f;
    scanf("%d%d%d", &n, &m, &f);
    int s = 0, t = 1;
    dinic.init(n);
    for (int i = 0, u, v, w; i < m; i++) {
      scanf("%d%d%d", &u, &v, &w);
      edge_id[i] = dinic.addEdge(v, u, w);
    }
    int sum = 0;
    for (int i = 0, u, w; i < f; i++) {
      scanf("%d%d", &u, &w);
      sum += w;
      dinic.addEdge(s, u, w);
    }
    int flow = dinic.MaxFlow(s, t);
    printf("Case %d: %d\n", kase++, sum - flow);

    dinic.findCut(m);
    printf("%d", dinic.cut.size());
    for (int i = 0; i < dinic.cut.size(); i++) {
      printf(" %d", dinic.cut[i]);
    }
    puts("");
  }
}
```