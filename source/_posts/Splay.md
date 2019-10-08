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

理解是关键，因为很多操作都需要自己实现。


## 函数说明

```
int sz[u]: 以u为根的子树的大小
int ch[u][0/1]: u的左/右儿子，值为0时表示没有该儿子。
int fa[u]: u的父亲
int key[u]: u点所代表的权值
int cnt[u]: u点的数量
int top: 栈顶（用于内存回收）
int rt: 根结点
int tot: 树的大小
```

注意：
fa[u] = 0 表示u为根结点
ch[u][0/1] = 0 表示没有相应的孩子
任何时候都不应该修改 0 这个结点的值

### newnode
作用：生成一个新结点并返回其编号。
参数：父亲结点编号，新结点的值。如果父亲结点编号为0，则该结点为根。
返回值：该结点编号。

内存回收：把删除的结点放到栈中，建立新节点的使用可以利用已经删除结点的编号。

### get
作用：判断该结点是其父亲的哪个节点。
参数：结点编号。
返回值：0 表示该结点是父亲的左儿子，1 表示该结点是父亲的右儿子。

单独使用时需要判父亲是否为0。

### rotate
作用：将该结点向祖先旋转一个高度。
参数：结点编号。
返回值：无

单独使用时需要判父亲是否为0。
对于splay操作来说，pushup(x)是多余的。

### 查询比某个数大（小）的第一个元素

先将val插入后，查询根的后继（前驱），最后删除 val 即可。但是常数比较大。

也可以手动寻找。


## 模板

```cpp

struct Splay {
  int fa[maxn], ch[maxn][2], stk[maxn];
  int sz[maxn], key[maxn], cnt[maxn];
  int rt, top, tot;

  // private

  int newnode(int p = 0, int k = 0) {
    int x = top ? stk[top--] : ++tot;
    if (p) ch[p][key[p] < k] = x;
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

  int get(int x) { return x == ch[fa[x]][1]; }

  void rotate(int x) {
    int y = fa[x], z = fa[y], d = get(x);
    if (z) ch[z][get(y)] = x;
    fa[x] = z, fa[y] = x;
    ch[y][d] = ch[x][d ^ 1], fa[ch[y][d]] = y;
    ch[x][d ^ 1] = y;
    pushup(y), pushup(x);
  }

  // pubilc

  void init() {
    rt = top = tot = 0;
    ch[rt][0] = ch[rt][1] = 0;
    cnt[rt] = sz[rt] = fa[rt] = 0;
  }

  // 旋到 goal 的下面
  void splay(int x, int goal = 0) {
    for (int f = fa[x]; f = fa[x], f != goal; rotate(x)) {
      if (fa[f] != goal) rotate(get(x) == get(f) ? f : x);
    }
    if (!goal) rt = x;
  }

  // 查询第k小的节点。 1-index
  // [1,2,2,3] kth(3) = 2
  int kth(int k) {
    for (int p = rt; p;) {
      if (k <= sz[ch[p][0]])
        p = ch[p][0];
      else {
        k -= sz[ch[p][0]] + cnt[p];
        if (k <= 0) {
          splay(p);
          return p;
        }
        p = ch[p][1];
      }
    }
    // k > 树的大小，或者 k < 1
    return 0;
  }

  // 查询值小于val的结点数量。 1-index
  // val不一定存在于Splay中
  int rank(int val) {
    int ret = 0;
    for (int p = rt; p;) {
      if (val < key[p])
        p = ch[p][0];
      else {
        ret += sz[ch[p][0]];
        if (val == key[p]) {
          splay(p);
          return ret;
        }
        ret += cnt[p];
        p = ch[p][1];
      }
    }
    return ret;
  }

  // 插入值为val的节点
  void insert(int val) {
    for (int p = rt, f = 0;; f = p, p = ch[p][key[p] < val]) {
      if (!p) {
        int x = newnode(f, val);
        pushup(x), pushup(f), splay(x);
        break;
      }
      if (key[p] == val) {
        cnt[p]++;
        pushup(p), pushup(f), splay(p);
        break;
      }
    }
  }

  // 前驱节点，若返回0，则表示无前驱
  int pre() {
    int p = ch[rt][0];
    while (ch[p][1]) p = ch[p][1];
    return p;
  }
  // 后继节点，若返回0，则表示无后继
  int nxt() {
    int p = ch[rt][1];
    while (ch[p][0]) p = ch[p][0];
    return p;
  }

  // 找到值为val的结点，并将其旋转到根
  bool find(int val) {
    for (int p = rt; p;) {
      if (key[p] == val) {
        splay(p);
        return true;
      }
      if (val < key[p])
        p = ch[p][0];
      else
        p = ch[p][1];
    }
    return false;
  }

  // 删除值为val的节点
  void remove(int val) {
    if (!find(val)) return;  // 若找到则将其旋转到根
    if (cnt[rt] > 1) {
      --cnt[rt], pushup(rt);
      return;
    }
    stk[++top] = rt;  // 内存回收
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

  // 找到第一个严格大于 val 的元素。
  int bigger(int val) {
    int ret = 1e9;
    for (int p = rt; p;) {
      if (key[p] > val) {
        ret = min(ret, key[p]);
        p = ch[p][0];
      } else
        p = ch[p][1];
    }
    return ret;
  }
} splay;
```


# Splay 维护区间

根据区间顺序进行建树，树型决定了偏序关系。与区间内元素的取值无关。

那么如果我们要处理区间 $[l,r]$ 的信息。我们可以通过先将 $l-1$ 旋转到根，再将 $r+1$ 旋转到根的儿子。这样 $r+1$ 左边这棵树就是区间 $[l,r]$。 为了不特判边界情况，我们可以在首尾各插入一个哨兵节点。 

比如注意哨兵节点，注意 $pushdown$ 和 $pushup$，注意修改的时候别忘了修改父亲。

## 代码
```cpp

#define key_value ch[ch[rt][1]][0]
int a[maxn];
struct Splay {
  int sz[maxn], ch[maxn][2], fa[maxn];
  int key[maxn], stk[maxn];
  bool rev[maxn];
  int rt, tot;

  void pushup(int x) { sz[x] = 1 + sz[ch[x][0]] + sz[ch[x][1]]; }
  void pushr(int x) {
    swap(ch[x][0], ch[x][1]);
    rev[x] ^= 1;
  }
  void pushdown(int x) {
    if (rev[x]) {
      if (ch[x][0]) pushr(ch[x][0]);
      if (ch[x][1]) pushr(ch[x][1]);
      rev[x] = 0;
    }
  }

  // 1-index
  void init(int n) {
    tot = top = 0;
    for (int i = 1; i <= n; i++) a[i] = i;
    // 首尾插入两个哨兵结点
    rt = newnode(0, -1);
    ch[rt][1] = newnode(rt, -1);
    key_value = build(1, n, ch[rt][1]);
    pushup(ch[rt][1]), pushup(rt);
  }

  int build(int l, int r, int p) {
    if (l > r) return 0;
    int m = l + r >> 1;
    int x = newnode(p, a[m]);
    ch[x][0] = build(l, m - 1, x);
    ch[x][1] = build(m + 1, r, x);
    pushup(x);
    return x;
  }

  int newnode(int p = 0, int k = 0) {
    int x = ++tot;
    fa[x] = p, sz[x] = 1, key[x] = k;
    ch[x][0] = ch[x][1] = 0;
    rev[x] = 0;
    return x;
  }

  int get(int x) { return x == ch[fa[x]][1]; }
  void rotate(int x) {
    int y = fa[x], z = fa[y], d = get(x);
    if (z) ch[z][get(y)] = x;
    fa[x] = z, fa[y] = x;
    ch[y][d] = ch[x][d ^ 1], fa[ch[y][d]] = y;
    ch[x][d ^ 1] = y;
    pushup(y), pushup(x);
  }
  void splay(int x, int goal = 0) {
    int top = 0;
    stk[++top] = x;
    for (int i = x; i != goal; i = fa[i]) stk[++top] = fa[i];
    for (int i = top; i; i--) pushdown(stk[i]);

    for (int f = fa[x]; f = fa[x], f != goal; rotate(x)) {
      if (fa[f] != goal) rotate(get(x) == get(f) ? f : x);
    }
    if (!goal) rt = x;
  }

  // 返回序列中第 k-1 个节点（有哨兵），1-index
  int kth(int r, int k) {
    pushdown(r);
    int t = sz[ch[r][0]] + 1;
    if (t == k) return r;
    return t > k ? kth(ch[r][0], k) : kth(ch[r][1], k - t);
  }

  // 选择区间 [l,r], key_value 为区间 [l,r] 的根节点
  void select(int l, int r) {
    splay(kth(rt, l), 0);
    splay(kth(ch[rt][1], r - l + 2), rt);
  }

  // 后继节点
  int nxt() {
    pushdown(rt);  // 记得 pushdown!
    int p = ch[rt][1];
    pushdown(p);
    while (ch[p][0]) {
      p = ch[p][0];
      pushdown(p);
    }
    return p;
  }

  // 反转区间 [l,r]
  void reverse(int l, int r) {
    select(l, r);
    pushr(key_value);
  }
  // 将区间[l,r]切下，移动到p后面。
  void cut(int l, int r, int p) {
    select(l, r);
    int tmp = key_value;
    fa[tmp] = 0;
    key_value = 0;

    pushup(ch[rt][1]), pushup(rt);
    splay(kth(rt, p + 1));
    int succ = nxt();
    ch[succ][0] = tmp;
    fa[tmp] = succ;
    pushup(succ);
    splay(succ);
  }

  // 输出树的时候记得 pushdown!
} splay;
```

