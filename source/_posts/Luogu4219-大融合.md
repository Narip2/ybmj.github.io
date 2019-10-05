---
title: Luogu4219 大融合
date: 2019-10-05 14:29:41
categories:
- ACM
- 题目
tags:
- LCT
---

[题目链接](https://www.luogu.org/problem/P4219)

# 题目

$n$ 个点，$m$ 个操作：
1. $(u,v)$ 连一条边
2. $(u,v)$ 查询经过这条边的简单路径数

$1 \leq n,m \leq 10^5$

# 分析

$LCT$ 维护子树大小。

对于操作 $2$，答案是 $u$ 的虚子树和 $v$ 的虚子树 的乘积。

# 代码
```cpp
#include <bits/stdc++.h>
using namespace std;
const int maxn = 1e5 + 5;

// 1-index
struct LCT {
  int val[maxn], sum[maxn], si[maxn];  // 基于点权
  int rev[maxn], ch[maxn][2], fa[maxn];
  int stk[maxn];

  inline bool isroot(int x) { return ch[fa[x]][0] != x && ch[fa[x]][1] != x; }
  inline bool get(int x) { return ch[fa[x]][1] == x; }
  inline void reverse(int x) {
    swap(ch[x][0], ch[x][1]);
    rev[x] ^= 1;
  }
  inline void pushdown(int x) {
    if (rev[x]) {
      if (ch[x][0]) reverse(ch[x][0]);
      if (ch[x][1]) reverse(ch[x][1]);
      rev[x] ^= 1;
    }
  }

  // 因为儿子可能会改变，因此每次必须重新计算
  inline void pushup(int x) {
    sum[x] = 1 + sum[ch[x][0]] + sum[ch[x][1]] + si[x];
  }

  // 避免单独使用：不能直接旋转根
  void rotate(int x) {
    int y = fa[x], z = fa[y], d = get(x);
    if (!isroot(y)) ch[z][get(y)] = x;
    fa[x] = z;
    ch[y][d] = ch[x][d ^ 1], fa[ch[y][d]] = y;
    ch[x][d ^ 1] = y, fa[y] = x;
    pushup(y), pushup(x);
  }
  // 将x旋转到Splay的根
  void splay(int x) {
    int top = 0;
    stk[++top] = x;
    for (int i = x; !isroot(i); i = fa[i]) stk[++top] = fa[i];
    for (int i = top; i; i--) pushdown(stk[i]);
    for (int f; !isroot(x); rotate(x))
      if (!isroot(f = fa[x])) rotate(get(x) == get(f) ? f : x);
  }

  // 将根到x的路径拉到一棵Splay中
  void access(int x) {
    for (int y = 0; x; y = x, x = fa[x]) {
      splay(x);
      si[x] += sum[ch[x][1]];
      ch[x][1] = y;
      si[x] -= sum[y];
      pushup(x);
    }
  }
  // 让x成为原树的根，x为深度最低的点，其左子树为空
  void makeroot(int x) { access(x), splay(x), reverse(x); }

  // 找x所在原树的根。主要用来判联通性，如果find(x) = find(y)
  // 则x,y在同一棵树中。
  int find(int x) {
    access(x), splay(x);
    while (ch[x][0]) pushdown(x), x = ch[x][0];
    splay(x);
    return x;
  }
  // 加边，y为深度最低的点（顺序无所谓）
  void link(int x, int y) {
    split(x, y);
    si[fa[x] = y] += sum[x];
    pushup(y);
  }
  // 删边
  void cut(int x, int y) { split(x, y), fa[x] = ch[y][0] = 0, pushup(y); }
  // 将x到y的路径拉到一棵Splay中，y为Splay的根
  void split(int x, int y) { makeroot(x), access(y), splay(y); }
  long long query(int x, int y) {
    split(x, y);
    return 1LL * (si[x] + 1) * (si[y] + 1);
  }
} lct;

int main() {
  int n, q;
  scanf("%d%d", &n, &q);
  char op[5];
  int x, y;
  while (q--) {
    scanf("%s%d%d", op, &x, &y);
    if (op[0] == 'A')
      lct.link(x, y);
    else
      printf("%lld\n", lct.query(x, y));
  }
}
```