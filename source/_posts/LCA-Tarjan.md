---
title: LCA---Tarjan
comments: true
date: 2018-03-28 11:23:45
categories:
  - ACM
  - 图论
tags:
  - 图论
  - LCA
---

网上看到一篇很详细的 blog，带有模拟过程，所以自己就不再写啦（懒癌..

Tarjan 算法是离线的，时间复杂度为 O(n+q)

[参考博客](https://www.cnblogs.com/JVxie/p/4854719.html)

```cpp
const int maxn = 1000000;
struct LCA {
    vector<int> G[maxn];
    vector<pii> Q[maxn];
    bool vis[maxn];
    int par[maxn], dist[maxn], ANS[maxn];

    int find(int u) { return u == par[u] ? u : par[u] = find(par[u]); }
    void merge(int u, int v) {
        u = find(u);
        v = find(v);
        if (u != v) {
            par[v] = u;
        }
    }

    void init(int n, int m) {
        for (int i = 0; i < n + 1; i++) {
            G[i].clear();
            vis[i] = false;
            par[i] = i;
        }
        for (int i = 0; i < m + 1; i++) Q[i].clear();
    }

    void add_edge(int u, int v) {
        G[u].pb(v);
        G[v].pb(u);
    }

    void add_query(int u, int v, int id) {
        Q[u].pb(mp(v, id));
        Q[v].pb(mp(u, id));
    }

    void Tarjan(int u, int fa) {
        vis[u] = true;
        for (auto v : G[u]) {
            if (fa != v) {
                Tarjan(v, u);
                merge(u, v);  // 注意合并顺序，一定是v合并到u上
            }
        }
        for (auto V : Q[u]) {
            if (vis[V.first] == true) {
                ANS[V.second] = find(V.first);  // 公共祖先为par[v]
            }
        }
    }
} lca;
```
