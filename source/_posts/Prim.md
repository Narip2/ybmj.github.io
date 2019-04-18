---
title: Prim
comments: true
date: 2018-04-04 11:20:36
categories:
  - ACM
  - 图论
tags:
  - 图论
  - 最小生成树
---

# Prim

**思想**:
Prim 和 Kruskal 实质上都是用了权值最小的边一定在生成树中这个性质。

Kruskal 对于稠密图来说性能比较差，因为要维护一个权值最小的边，所以有一个优先队列。

而 Prim 对于稠密图有比较好的性能，那么它是怎么确定权值最小的边？

Prim 与 Kruskal 不同的是，Prim 每次以一个没有访问过的点为起点，在所有连接着起点的边中找一个最小。而 Kruskal 每次找的是整张图中权值最小的边，并用并查集来维护访问过的点。

设 n 为顶点数量

所以 Prim 只要对每个顶点都做一次上述的操作即可得到最小生成树，时间复杂度为$O(n^2)$

```cpp
// 耗费矩阵cost[][],标号从0开始,0~n-1
// 返回最小生成树的权值,返回-1表示原图不连通
// O(n^2)
const int maxn = 100;
bool vis[maxn];
int lowc[maxn];
int cost[maxn][maxn];  //初始化为正无穷

int Prim(int n) {
    int ans = 0;
    clr(vis, 0);
    vis[0] = 1;
    for (int i = 1; i < n; i++) lowc[i] = cost[0][i];
    for (int i = 1; i < n; i++) {
        int minc = INF;
        int p = -1;
        for (int j = 0; j < n; j++)
            if (!vis[j] && minc > lowc[j]) {
                minc = lowc[j];
                p = j;
            }
        if (minc == INF) return -1;
        vis[p] = 1;
        ans += minc;
        for (int j = 0; j < n; j++)
            if (!vis[j] && lowc[j] > cost[p][j]) lowc[j] = cost[p][j];
    }
    return ans;
}
```
