---
title: CDQ分治
date: 2019-11-11 17:19:43
categories:
- ACM
- 分治
tags:
- CDQ分治
---

# CDQ分治

对于区间$[l,r)$，先分治计算 $[l,m), [m,r)$，然后再计算 $[l,m)$ 中的元素对 $[m,r)$ 的贡献。

需要注意的就是右边的区间不会对左边的区间产生贡献，否则无法CDQ分治。


以 [luogu3810](https://www.luogu.org/problem/P3810) 为例。

有 $n$ 个元素，第 $i$ 个元素有 $a_i,b_i,c_i$ 三个属性，设 $f(i)$ 表示满足 $ a_j \leq a_i, b_j \leq b_i, c_j \leq c_i $ 的 $j$ 的数量。对于 $d \in [0, n)$，求 $f(i) = d$ 的数量。

首先按 $a$ 排序后就能保证右边的不会对左边产生贡献。 然后开始分治。

对于区间 $[l,r)$，先分治 $[l,m), [m,r)$。 然后计算区间 $[l,m)$ 对 区间 $[m,r)$ 的贡献。

首先可以把 $[l,m)$ 和 $[m,r)$ 按照 $b$ 排序，然后维护一个关于 $c$ 的树状数组。每次区间查询即可。

需要注意的就是对于两个相同的元素，他们相互之间是贡献的，因此第一步要先去重。
```cpp
#include <bits/stdc++.h>
using namespace std;
const int maxn = 2e5 + 5;
using ll = long long;
struct Node {
  int a, b, c, id, num, ans;
  bool operator==(const Node &rhs) const {
    return (a == rhs.a && b == rhs.b && c == rhs.c);
  }
} a[maxn];
int ans[maxn], sum[maxn];
int n, k;

inline void add(int p, int x) {
  for (int i = p; i <= k; i += i & -i) sum[i] += x;
}
inline int query(int l, int r) {
  int ret = 0;
  for (int i = r; i > 0; i -= i & -i) ret += sum[i];
  for (int i = l - 1; i > 0; i -= i & -i) ret -= sum[i];
  return ret;
}

void solve(int l, int r) {
  if (r - l <= 1) return;
  int m = l + r >> 1;
  solve(l, m);
  solve(m, r);
  int p = l;
  for (int i = m; i < r; i++) {
    while (p < m && a[p].b <= a[i].b) add(a[p].c, a[p].num), p++;
    a[i].ans += query(1, a[i].c);
  }
  for (int i = l; i < p; i++) add(a[i].c, -a[i].num);
  sort(a + l, a + r, [&](const Node &A, const Node &B) {
    if (A.b == B.b) return A.c < B.c;
    return A.b < B.b;
  });
}
int main() {
  scanf("%d%d", &n, &k);
  for (int i = 0; i < n; i++)
    scanf("%d%d%d", &a[i].a, &a[i].b, &a[i].c), a[i].id = i, a[i].num = 1,
                                                a[i].ans = 0;
  sort(a, a + n, [&](const Node &A, const Node &B) {
    if (A.a == B.a && A.b == B.b) return A.c < B.c;
    if (A.a == B.a) return A.b < B.b;
    return A.a < B.a;
  });
  int m = 0;
  for (int i = 1; i < n; i++) {
    if (a[i] == a[m])
      a[m].num++;
    else
      a[++m] = a[i];
  }
  solve(0, m + 1);
  for (int i = 0; i <= m; i++) ans[a[i].ans + a[i].num - 1] += a[i].num;
  for (int i = 0; i < n; i++) printf("%d\n", ans[i]);
}
```