---
title: cf102059D Dumae
date: 2019-08-26 21:02:56
categories:
- ACM
- 题目
tags:
- 贪心
---

# 题意
$n$ 个人排队，每个人有一个要求的区间，同时右 $m$ 个关系表示他们的偏序。求一种可行的方案。


# 分析

首先判断给出的 $m$ 个二元关系是否是偏序关系。然后在逆拓扑序上 DP 求出每个人最大的位置，$\displaystyle R_u = \min_{(u, v) \in e} \{R_v - 1\}$

接着在进行一次拓扑排序，使用两个优先队列，第一个队列存放所有度为零的点，当某个点的 $L_u \leq i$ 时，丢进第二个优先队列中（满足 $L_u$ 的限制），从第二个优先队列中贪心地选取 $R_u$ 最小的点放在当前位置，并删边。

# 代码

```cpp
#include <bits/stdc++.h>
using namespace std;
const int maxn = 3e5 + 5;
int L[maxn], R[maxn], d[maxn];
vector<int> G[maxn];
bool vis[maxn];
 
int n, m;
bool topo() {
  queue<int> q;
  for (int i = 1; i <= n; i++)
    if (d[i] == 0) q.push(i);
  int cnt = q.size();
  while (!q.empty()) {
    int u = q.front();
    q.pop();
    for (auto &v : G[u]) {
      d[v]--;
      if (d[v] == 0) {
        q.push(v);
        cnt++;
      }
    }
  }
  return cnt == n;
}
int dfs(int u) {
  if (vis[u]) return R[u];
  vis[u] = 1;
  for (auto &v : G[u]) R[u] = min(R[u], dfs(v) - 1);
  return R[u];
}
int ans[maxn], dd[maxn];
using pii = pair<int, int>;
bool solve() {
  priority_queue<pii, vector<pii>, greater<pii>> pq[2];
  for (int i = 1; i <= n; i++)
    if (!dd[i]) pq[0].push({L[i], i});
  int id = 1;
  while (!pq[0].empty() || !pq[1].empty()) {
    while (true) {
      if (pq[0].size() == 0) break;
      auto it = pq[0].top();
      int u = it.second;
      if (L[u] > id) break;
      pq[0].pop();
      pq[1].push({R[u], u});
    }
    if (pq[1].empty()) break;
    if (!pq[1].empty()) {
      auto it = pq[1].top();
      pq[1].pop();
      int u = it.second;
      if (R[u] < id) return false;
      ans[id++] = u;
      for (auto &v : G[u]) {
        dd[v]--;
        if (dd[v] == 0) {
          pq[0].push({L[v], v});
        }
      }
    }
  }
  return id == n + 1;
}
int main() {
  scanf("%d%d", &n, &m);
  for (int i = 1; i <= n; i++) scanf("%d%d", L + i, R + i);
  for (int i = 0, u, v; i < m; i++) {
    scanf("%d%d", &u, &v);
    G[u].push_back(v);
    d[v]++;
    dd[v]++;
  }
  if (topo() == false) {
    puts("-1");
    return 0;
  }
  for (int i = 1; i <= n; i++) R[i] = dfs(i);
  bool ok = solve();
  if (!ok)
    puts("-1");
  else
    for (int i = 1; i <= n; i++) printf("%d\n", ans[i]);
}
```
