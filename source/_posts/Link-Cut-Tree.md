---
title: Link Cut Tree
date: 2019-09-30 15:50:40
categories:
- ACM
- 数据结构
tags:
- LCT
---

[参考博客](https://www.cnblogs.com/flashhu/p/8324551.html)

LCT 维护了一个Splay森林。

每个Splay维护的是一条深度递增的路径。

可以维护树链的信息，支持换根，增删边。

边权模型：对每个边建立一个点，该点的点权就是边权。

```cpp

// 1-index
struct LCT {
  int val[maxn], sum[maxn];  // 基于点权
  int rev[maxn], ch[maxn][2], fa[maxn];
  int stk[maxn];
  inline void init(int n) {  // 初始化点权（一般不需要）
    for (int i = 1; i <= n; i++) scanf("%d", val + i);
    for (int i = 1; i <= n; i++) fa[i] = ch[i][0] = ch[i][1] = rev[i] = 0;
  }

  inline bool isroot(int x) { return ch[fa[x]][0] != x && ch[fa[x]][1] != x; }
  inline bool get(int x) { return ch[fa[x]][1] == x; }
  inline void reverse(int x) {
    swap(ch[x][0], ch[x][1]);
    rev[x] ^= 1;
  }
  inline void pushdown(int x) {
    if (!rev[x]) return;
    if (ch[x][0]) reverse(ch[x][0]);
    if (ch[x][1]) reverse(ch[x][1]);
    rev[x] ^= 1;
  }

  // 因为儿子可能会改变，因此每次必须重新计算
  inline void pushup(int x) { sum[x] = val[x] + sum[ch[x][0]] + sum[ch[x][1]]; }

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
    // 父亲不认儿子，但儿子认父亲
    for (int y = 0; x; y = x, x = fa[x]) splay(x), ch[x][1] = y, pushup(x);
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
    makeroot(x);
    // if(find(y) != x) fa[x] = y; // 存在不合法加边
    fa[x] = y, splay(x);
  }
  // 删边
  void cut(int x, int y) {
    split(x, y), fa[x] = ch[y][0] = 0, pushup(y);
    /*
      存在不合法删边
      makeroot(x);
      if(find(y) != x || fa[y] != x || ch[y][0]) return false;
      fa[y] = ch[x][1] = 0;
      pushup(x);
      return true;
    */
  }
  // 将x到y的路径拉到一棵Splay中，y为Splay的根
  void split(int x, int y) { makeroot(x), access(y), splay(y); }

  // 单点修改
  void update(int x, int v) {
    access(x), splay(x);
    val[x] = v;
    pushup(x);
  }

  // 区间修改
  void update_seg(int u, int v) {
    split(u, v);
    tag(v); // 打对应标记
  }
} lct;
```