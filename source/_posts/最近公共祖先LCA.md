---
title: 最近公共祖先LCA
date: 2019-08-01 22:22:35
categories:
  - ACM
  - 图论
tags:
  - 图论
  - LCA
---

## 倍增法

可以在树上直接倍增求 $LCA$，查询复杂度是 $O(\log (n))$

经常在倍增时维护其他信息。

```cpp
const int BASE = 20;
int dep[maxn];  // 深度
int par[maxn][BASE + 1];
void dfs(int u, int fa) {
  dep[u] = dep[fa] + 1;
  par[u][0] = fa;
  for (auto& v : G[u]) {
    if (v == fa) continue;
    dfs(v, u);
  }
}
void init(int n) {
  for (int j = 1; j <= BASE; j++)
    for (int i = 1; i <= n; i++) {
      int v = par[i][j - 1];
      par[i][j] = par[v][j - 1];
    }
}

int lca(int u, int v) {
  if (dep[u] > dep[v]) swap(u, v);
  for (int i = BASE; i >= 0; i--)
    if (dep[par[v][i]] >= dep[u]) v = par[v][i];  // 先跳到同一高度

  if (u == v) return u;
  for (int i = BASE; i >= 0; i--)
    if (par[u][i] != par[v][i]) u = par[u][i], v = par[v][i];  // 一起向上跳
  return par[u][0];
}
```


## ST表法

将树型转化为区间形式，然后再 $RMQ$ 所查区间内深度最小的点，即为 $lca$


![](https://ybmj-blog-1256173108.cos.ap-shanghai.myqcloud.com/blog-picture/QQ%E5%9B%BE%E7%89%8720190731223849.jpg)


查询 $2$ 与 $3$ 的 $lca$ 时，会查询其 $dfs$ 序上区间 $dfn$ 的最小值。


```cpp
// 预处理 O(nlogn), 查询 O(1)
const int maxn = 1e6+5;
struct LCA {
    vector<int> G[maxn], sp;
    int dep[maxn], dfn[maxn];
    pii dp[21][maxn << 1];
    void init(int n) {
        for (int i = 0; i <= n; i++) G[i].clear();
        sp.clear();
    }
    void addEdge(int u, int v) {
        G[u].push_back(v);
        G[v].push_back(u);
    }
    void dfs(int u, int fa) {
        dep[u] = dep[fa] + 1;
        dfn[u] = sp.size();
        sp.push_back(u);
        for (auto &v : G[u]) {
            if (v == fa) continue;
            dfs(v, u);
            sp.push_back(u);
        }
    }
    void initRmq() {
        int n = sp.size();
        for (int i = 0; i < n; i++) dp[0][i] = {dfn[sp[i]], sp[i]};
        for (int i = 1; (1 << i) <= n; i++)
            for (int j = 0; j + (1 << i) - 1 < n; j++)
                dp[i][j] = min(dp[i - 1][j], dp[i - 1][j + (1 << (i - 1))]);
    }
    int lca(int u, int v) {
        int l = dfn[u], r = dfn[v];
        if (l > r) swap(l, r);
        int k = 31 - __builtin_clz(r - l + 1);
        return min(dp[k][l], dp[k][r - (1 << k) + 1]).second;
    }
} lca;
```

## Tarjan

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
