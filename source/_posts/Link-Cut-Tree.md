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

# LCT
$LCT$ 通过维护一个 $Splay$ 森林，以起到维护一棵树的信息的作用。

在 $LCT$ 中，一棵树的边被分为实边（偏爱边）和虚边，每个实边连接的连通块（子树）用一个 $Splay$ 来维护。这个 $Splay$ 中的信息仅仅是这个由实边组成的子树的信息，不会包含子树外的点。

## 实边和虚边的定义

因为实边和 $Splay$ 是紧密相关的，所以我们不妨先看看这些 $Splay$ 的特点。

每个 $Splay$ 中的**偏序关系**是点在原树中的**深度**，且每个 $Splay$ 中无深度相同的两个点。每个 $Spaly$ 的中序遍历，在原树上是一条**深度递增的树链**（非常重要）。

那也就要求了每个父结点和它的儿子结点之间**最多**只有一条实边，其余都是虚边。

每条实边都只属于一个 $Splay$。

## 如何用 $Splay$ 森林维护一棵树的信息（不同 $Splay$ 之间是如何联系的？）

所有的**虚边**实际上都是一棵 $Splay$ 中 深度最低的结点 的父亲，指向另一颗（深度更低）$Splay$ 的某个结点。

但对于一棵 $Splay$ 来说，它不可能通过查找儿子而进入到另一颗 $Splay$ 中。换句话说，只能通过向祖先寻找的方式，进入到另一棵（深度更低）$Splay$ 中。这就是为什么可以做到每棵 $Splay$ 只维护了当前子树的信息，而不涉及其它的结点。

## LCT 的灵魂

### 虚边和实边的转换
我们可以通过将结点 $x$ 到根的路径的边都设置为实边（偏爱路径），这样这条链的信息就在一棵 $Splay$ 中，我们可以快速地维护这条树链上的信息。

我们可以用 `access(x)` 来表示这一操作。

具体做法：
1. 先将结点 $x$ 旋转到当前 $Splay$ 的根，然后它的右儿子（深度较大）是不在根到 $x$ 的这条树链上的，因此要置空。但此时其右儿子的祖先还是 $x$ ，这条边也就变成了虚边。
2. 然后寻找祖先 $y = fa[x]$，将 $y$ 旋转到它所在 $Splay$ 的根后，再将右儿子置为 $x$。因为 $y$ 的深度是 $x$ 所在 $Splay$ 的最低深度 减一，所以 $y$ 的右儿子必然不在 $x$ 到根的这条树链上。
3. 不断重复 $2$ 操作，直到 $y$ 的父亲为空。

### 换根
通过之前的操作，我们可以很容易的维护一条根到任意结点的树链信息。但如果我们想维护一条普通的树链（端点可能不为根），这时候就需要换根操作。

我们可以用 `makeroot(x)` 来表示这一操作。

具体做法：
1. `access(x)`
2. 将 $x$ 转到 $Splay$ 的根.
3. 将 $x$ 的儿子互换。因为 $x$ 是 $Splay$ 中深度最大的结点，因此此时 $x$ 的右儿子为空。而根结点一定是深度最低的点，所以我们只要将 $x$ 的儿子互换，$x$ 就成为了深度最低的点，即原树的根。

## 其它操作

### Split

`split(u,v)` 将树链 $(u,v)$ 放到一个 $Splay$ 中。

具体做法：
1. `makeroot(u)`
2. `access(v)`
3. 将 $v$ 转到当前 $Splay$ 的根。这样 $v$ 点的信息就是整棵 $Splay$，即 $(u,v)$ 的信息。

### 插入

插入的顺序其实没有关系，因为树的结构是固定的，想要谁当根只需要做一下换根操作即可。

### 区间更新

对于树链 $(u,v)$，先要通过 `split(u,v)` 将 $(u,v)$ 链上的点都放到一个 $Splay$ 中。

然后对 $v$ 打上对应的标记即可。

### 单点更新

要注意因为某些区间标记（比如区间取反）会影响点权，因此我们必须在更新之前先把区间标记都做掉。

具体做法：
1. `access(x)`
2. 将 $x$ 旋转到当前 $Splay$ 的根
3. 更新 $x$ 的值，同时 $pushup(x)$


## LCT的作用：

1. 维护树链的信息
2. 动态维护树的连通性：因为LCT本质是个Splay森林，所以可能存在不连通的情况。可以通过找各自原树的根是否相同来判断是否连通。（有些卡常题目可以用并查集来替代 $find$ 判联通性）

## 时间复杂度：

势能分析据说 $O((n + m) \log n)$，常数血大。


## 边权模型：

对于每条边建一个点，这个点的权值即为边权。


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
    if (rev[x]) {
      if (ch[x][0]) reverse(ch[x][0]);
      if (ch[x][1]) reverse(ch[x][1]);
      rev[x] ^= 1;
    }
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
    // tag(v); 打对应标记，记得更改对sum的影响
  }
} lct;
```