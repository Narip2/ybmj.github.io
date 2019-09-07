---
title: Splay
date: 2019-09-05 16:37:16
categories:
- ACM
- 数据结构
tags:
- 平衡树
---

# Splay

每次访问一个节点后，将该节点旋转到根。均摊下来每次操作的复杂度是 $O(\log(n))$。

## 代码

```cpp
struct Splay {
  int sz[maxn], ch[maxn][2], fa[maxn];
  int key[maxn], cnt[maxn];
  int rt, top, tot;
  int stk[maxn];  // 模拟内存

  // 判断x是其父亲的哪个节点
  inline int son(int x) { return x == ch[fa[x]][1]; }
  void init() {
    rt = top = tot = 0;
    ch[rt][0] = ch[rt][1] = 0;
    cnt[rt] = sz[rt] = fa[rt] = 0;
  }

  void rotate(int x) {
    int y = fa[x], z = fa[y], chk = son(x);
    ch[y][chk] = ch[x][chk ^ 1];
    fa[ch[x][chk ^ 1]] = y;
    ch[x][chk ^ 1] = y;
    fa[y] = x;
    fa[x] = z;
    if (z) ch[z][y == ch[z][1]] = x;  // !
    pushup(y), pushup(x);
  }

  // 旋到 goal 的下面
  void splay(int x, int goal = 0) {
    while (fa[x] != goal) {
      if (fa[fa[x]] == goal)
        rotate(x);
      else {
        int y = fa[x];
        int d = ch[fa[y]][0] == y;
        if (ch[y][d] == x)
          rotate(x);
        else
          rotate(y);
        rotate(x);
      }
    }
    pushup(x);
    if (!goal) rt = x;
  }

  int newnode(int p = 0, int k = 0) {
    int x = top ? stk[top--] : ++tot;
    if (p) ch[p][key[p] < k] = x;  // !
    fa[x] = p, sz[x] = 1, key[x] = k, cnt[x] = 1;
    ch[x][0] = ch[x][1] = 0;
    return x;
  }
  void pushup(int x) {
    if (!x) return;
    sz[x] = cnt[x];
    if (ch[x][0]) sz[x] += sz[ch[x][0]];
    if (ch[x][1]) sz[x] += sz[ch[x][1]];
  }

  // 查询第k小的节点。 1-index
  int kth(int k) {
    for (int p = rt;;) {
      if (ch[p][0] && k <= sz[ch[p][0]])
        p = ch[p][0];
      else {
        k -= cnt[p] + sz[ch[p][0]];
        if (k <= 0) {
          // 外部单独调用需要splay，其它情况如果不想改变树的形状就不要splay。
          splay(p);   
          return p;
        }
        p = ch[p][1];
      }
    }
  }

  // 查询值为val的节点是第几小的。 1-index
  int rank(int val) {
    int ret = 0;
    for (int p = rt;;) {
      if (val < key[p])
        p = ch[p][0];
      else {
        ret += sz[ch[p][0]];
        if (val == key[p]) {
          splay(p);
          return ret + 1;
        }
        ret += cnt[p];
        p = ch[p][1];
      }
    }
  }

  // 插入值为val的节点
  void insert(int val) {
    for (int p = rt, f = 0;;) {
      if (!p) {
        int x = newnode(f, val);
        pushup(x), pushup(f);
        splay(x);
        break;
      }
      if (key[p] == val) {
        cnt[p]++;
        pushup(p), pushup(f);
        splay(p);
        break;
      }
      f = p;
      p = ch[p][key[p] < val];
    }
  }

  // 前驱节点
  int pre() {
    int p = ch[rt][0];
    while (ch[p][1]) p = ch[p][1];
    return p;
  }
  // 后继节点
  int nxt() {
    int p = ch[rt][1];
    while (ch[p][0]) p = ch[p][0];
    return p;
  }

  // 删除值为val的节点
  void remove(int val) {
    rank(val);
    if (cnt[rt] > 1) {
      cnt[rt]--;
      pushup(rt);
      return;
    }
    stk[++top] = rt;
    if (!ch[rt][0] && !ch[rt][1]) {
      rt = 0;
      return;
    }
    if (!ch[rt][0]) {
      rt = ch[rt][1];
      fa[rt] = 0;
      return;
    }
    if (!ch[rt][1]) {
      rt = ch[rt][0];
      fa[rt] = 0;
      return;
    }
    int x = pre(), p = rt;
    splay(x);
    fa[ch[p][1]] = x;
    ch[x][1] = ch[p][1];
    pushup(rt);
  }
  
  // 求大于（小于）val的值：先将val插入后，查询根的后继（前驱），最后删除 val 即可。
} splay;
```


# Splay 维护区间

平衡树每个节点的权值表示区间的下标。

那么如果我们要处理区间 $[l,r]$ 的信息。我们可以通过先将 $l-1$ 旋转到根，再将 $r+1$ 旋转到根的儿子。这样 $r+1$ 左边这棵树就是区间 $[l,r]$。 为了不特判边界情况，我们可以在首尾各插入一个哨兵节点。 
## 代码
```cpp
#define key_value ch[ch[root][1]][0]
// 维护区间，平衡树维护下标
struct Splay {
  int a[maxn];
  int sz[maxn], ch[maxn][2], fa[maxn];
  int key[maxn], rev[maxn];
  int root, tot;
  int stk[maxn], top;

  // d = 0 左旋
  void rotate(int x, int d) {
    int y = fa[x];
    pushdown(y), pushdown(x);
    ch[y][d ^ 1] = ch[x][d];
    fa[ch[x][d]] = y;
    if (fa[y]) ch[fa[y]][ch[fa[y]][1] == y] = x;
    fa[x] = fa[y];
    ch[x][d] = y;
    fa[y] = x;
    pushup(y);
  }

  void splay(int x, int goal = 0) {
    // 旋到 goal 的下面
    pushdown(x);
    while (fa[x] != goal) {
      if (fa[fa[x]] == goal)
        rotate(x, ch[fa[x]][0] == x);
      else {
        int y = fa[x];
        int d = ch[fa[y]][0] == y;
        if (ch[y][d] == x)
          rotate(x, d ^ 1);
        else
          rotate(y, d);
        rotate(x, d);
      }
    }
    pushup(x);
    if (!goal) root = x;
  }

  // 返回第k小的节点
  int kth(int r, int k) {
    pushdown(r);
    int t = sz[ch[r][0]] + 1;
    if (t == k) return r;
    return t > k ? kth(ch[r][0], k) : kth(ch[r][1], k - t);
  }

  void select(int l, int r) {
    splay(kth(root, l), 0);
    splay(kth(ch[root][1], r - l + 2), root);
    // splay(kth(root, r + 2), root);  // 首部有个哨兵
    rev[key_value] ^= 1;
  }

  void pushup(int x) {
    sz[x] = 1;
    if (ch[x][0]) sz[x] += sz[ch[x][0]];
    if (ch[x][1]) sz[x] += sz[ch[x][1]];
  }

  // 区间反转
  void pushdown(int x) {
    if (!rev[x]) return;
    swap(ch[x][0], ch[x][1]);
    if (ch[x][0]) rev[ch[x][0]] ^= 1;
    if (ch[x][1]) rev[ch[x][1]] ^= 1;
    rev[x] = 0;
  }

  int build(int l, int r, int p) {
    if (l > r) return 0;
    int m = l + r >> 1;
    int x = newnode(p, a[m]);  // a[i]
    ch[x][0] = build(l, m - 1, x);
    ch[x][1] = build(m + 1, r, x);
    pushup(x);
    return x;
  }

  int newnode(int p = 0, int k = 0) {
    int x = top ? stk[top--] : ++tot;
    fa[x] = p, sz[x] = 1, key[x] = k;
    ch[x][0] = ch[x][1] = 0;
    rev[x] = 0;
    return x;
  }

  // 1-index
  void init(int n) {
    tot = top = 0;
    for (int i = 1; i <= n; i++) a[i] = i;
    // 首尾插入两个元素！！！
    root = newnode(0, -1);
    ch[root][1] = newnode(root, -1);
    key_value = build(1, n, ch[root][1]);
    pushup(ch[root][1]), pushup(root);
  }
} splay;
```