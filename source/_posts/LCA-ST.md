---
title: LCA-ST
comments: true
date: 2018-07-03 19:57:42
categories:
  - ACM
  - 图论
tags:
  - 图论
  - LCA
---

# ST

这是一种在线计算 $LCA$ 的算法，时间复杂度为： 预处理 $O(n \log n)$, 查询 $O(\log n)$

## 思想

将树型转化为区间形式，然后再 $RMQ$ 所查区间内深度最小的点，即为 $lca$


![](https://ybmj-blog-1256173108.cos.ap-shanghai.myqcloud.com/blog-picture/QQ%E5%9B%BE%E7%89%8720190731223849.jpg)


查询 $2$ 与 $3$ 的 $lca$ 时，会查询其 $dfs$ 序上区间 $dfn$ 的最小值。

## 模板

```cpp
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
