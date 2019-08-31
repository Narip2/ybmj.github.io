---
title: cf102222L Continuous Intervals
date: 2019-08-31 16:41:53
categories:
- ACM
- 题目
tags:
- 线段树
- 单调栈
---

[题目链接](https://codeforces.com/gym/102222/problem/L)

# 题意
给一个长度为 $n$ 的序列 $a$。 问有多少个区间 $[l,r]$ 满足：排序后，相邻两个数之间的差小于等于 $1$。

$1 \leq n \leq 1e5$

# 分析

首先考虑这个条件怎么用数学的方式表示：设 $cnt$ 为区间内的种类数，那么当 $max - min + 1 = cnt$ 满足时，当前区间就是满足条件的。

而且我们可以发现 $max - min + 1 >= cnt$，所以我们可以枚举右端点，对每个左端点维护 $max - min - cnt$ 的最小值以及相应的数量。

对于 $cnt$：我们只要预处理每个数上一个出现的位置即可，然后做一个区间减法。

对于 $max,min$：我们可以利用单调栈的思想，每次区间加/减一个最值的增量。这样对于每一个右端点都可以处理出来所有后缀的最值。

# 代码

```cpp
#include <bits/stdc++.h>
using namespace std;
#define lson (rt << 1)
#define rson (rt << 1 | 1)
#define Lson l, m, lson
#define Rson m + 1, r, rson
using ll = long long;
const int maxn = 1e5 + 5;

struct Node {
  ll sum, cnt;
  ll lazy;
} seg[maxn << 2];

int a[maxn];
int n;
int last[maxn];

inline void pushup(int rt) {
  auto &o = seg[rt], &ls = seg[lson], &rs = seg[rson];
  o.sum = min(ls.sum, rs.sum);
  o.cnt = 0;
  if (o.sum == ls.sum) o.cnt += ls.cnt;
  if (o.sum == rs.sum) o.cnt += rs.cnt;
}
inline void pushdown(int rt) {
  auto &o = seg[rt], &ls = seg[lson], &rs = seg[rson];
  if (!o.lazy) return;
  ls.sum += o.lazy;
  rs.sum += o.lazy;
  ls.lazy += o.lazy;
  rs.lazy += o.lazy;
  o.lazy = 0;
}
void build(int l, int r, int rt) {
  seg[rt].lazy = 0;
  if (l == r) {
    seg[rt].sum = 0;
    seg[rt].cnt = 1;
    return;
  }
  int m = l + r >> 1;
  build(Lson);
  build(Rson);
  pushup(rt);
}

void update(int l, int r, int rt, int L, int R, int x) {
  if (L > R || x == 0) return;
  if (L <= l && R >= r) {
    seg[rt].sum += x;
    seg[rt].lazy += x;
    return;
  }
  pushdown(rt);
  int m = l + r >> 1;
  if (L <= m) update(Lson, L, R, x);
  if (m + 1 <= R) update(Rson, L, R, x);
  pushup(rt);
}
ll query() {
  if (seg[1].sum == -1) return seg[1].cnt;
  return 0;
}

ll work() {
  ll ret = 0;
  map<int, int> tmp;
  for (int i = 1; i <= n; i++) {
    if (tmp.find(a[i]) == tmp.end())
      last[i] = 1;
    else
      last[i] = tmp[a[i]] + 1;
    tmp[a[i]] = i;
  }
  stack<int> mx, mi;
  for (int i = 1; i <= n; i++) {
    update(1, n, 1, last[i], i, -1);
    int r = i - 1;
    int MAX = a[i];
    while (!mx.empty() && a[i] > a[mx.top()]) {
      int p = mx.top();
      mx.pop();
      int L;
      if (mx.empty())
        L = 1;
      else
        L = mx.top() + 1;
      update(1, n, 1, L, r, a[i] - a[p]);
      if (!mx.empty()) r = mx.top();
    }
    mx.push(i);

    r = i - 1;
    int MIN = a[i];
    while (!mi.empty() && a[i] < a[mi.top()]) {
      int p = mi.top();
      mi.pop();
      int L;
      if (mi.empty())
        L = 1;
      else
        L = mi.top() + 1;
      update(1, n, 1, L, r, a[p] - a[i]);
      if (!mi.empty()) r = mi.top();
    }
    mi.push(i);

    ret += query();
  }

  return ret;
}

int main() {
  int T, kase = 1;
  scanf("%d", &T);
  while (T--) {
    scanf("%d", &n);
    for (int i = 1; i <= n; i++) scanf("%d", a + i);
    build(1, n, 1);
    ll ans = work();
    printf("Case #%d: %lld\n", kase++, ans);
  }
}
```