---
title: hdu3436 Queue-jumpers
date: 2019-09-09 14:00:28
categories:
- ACM
- 题目
tags:
- 平衡树
---

# 题意

$1$ 到 $n$ 顺序排列的数组，有 $m$ 次操作：
1. 将某个值为 $x$ 的移动到第一个位置
2. 查询值为 $x$ 的位置
3. 查询位置 $x$ 的值

$1 \leq n \leq 1e8, 1 \leq m \leq 1e5$

# 分析

看到 $n$ 的范围是 $1e8$，就感觉这题需要离线之后离散化区间了。

每个区间记录一个首元素的值和长度。

对于第一个操作：我们需要找到对应元素，然后将他删除，再放到首节点。

对于第二个操作：将值为 $x$ 的节点旋转到根，左子树的大小就是答案。

对于第三个操作：从根开始查询，根据 $sz$ 来判断应该走哪个儿子。

# 代码
```cpp
#include <bits/stdc++.h>
using namespace std;
#define my_unique(a) a.resize(distance(a.begin(), unique(a.begin(), a.end())))
#define my_sort_unique(c) (sort(c.begin(), c.end())), my_unique(c)
using pii = pair<int, int>;
const int maxn = 2e5 + 5;
int query[maxn];
vector<pii> a;
#define key_value ch[ch[rt][1]][0]
struct Splay {
  int sz[maxn], ch[maxn][2], fa[maxn];
  pii key[maxn];
  int rt, tot;
  int stk[maxn], top;
  int rev[maxn];  // 区间翻转标记
  map<int, int> pos;

  // 1-index
  void init() {
    pos.clear();
    tot = top = 0;
    // 首尾插入两个元素！！！
    rt = newnode(0, {-1, 0});
    ch[rt][1] = newnode(rt, {-1, 0});
    key_value = build(0, a.size() - 1, ch[rt][1]);
    pushup(ch[rt][1]), pushup(rt);
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

  int newnode(int p, pii k) {
    int x = top ? stk[top--] : ++tot;
    fa[x] = p, sz[x] = 1, key[x] = k;
    ch[x][0] = ch[x][1] = 0;
    pos[k.first] = x;
    return x;
  }

  // d = 0 左旋
  void rotate(int x, int d) {
    int y = fa[x];
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
    if (!goal) rt = x;
  }

  void pushup(int x) {
    sz[x] = key[x].second;
    if (ch[x][0]) sz[x] += sz[ch[x][0]];
    if (ch[x][1]) sz[x] += sz[ch[x][1]];
  }

  int pre() {
    int p = ch[rt][0];
    while (ch[p][1]) p = ch[p][1];
    return p;
  }
  int nxt() {
    int p = ch[rt][1];
    while (ch[p][0]) p = ch[p][0];
    return p;
  }
  void remove(int p) {
    splay(p);
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
    int x = pre();
    splay(x);
    fa[ch[p][1]] = x;
    ch[x][1] = ch[p][1];
    pushup(rt);
  }
  void Top(int w) {
    int p = pos[w];
    pii tmp = key[p];
    remove(p);

    splay(1);
    int x = nxt();
    ch[x][0] = newnode(x, tmp);
    pushup(x);
    splay(ch[x][0]);
  }

  void query(int x) {
    splay(pos[x]);
    printf("%d\n", sz[ch[rt][0]] + 1);
  }
  void rank(int x) {
    for (int p = rt; p;) {
      if (x <= sz[ch[p][0]])
        p = ch[p][0];
      else {
        x -= sz[ch[p][0]] + key[p].second;
        if (x <= 0) {
          printf("%d\n", key[p].first + x + key[p].second - 1);
          return;
        }
        p = ch[p][1];
      }
    }
  }
  void debug(int u) {
    if (ch[u][0]) debug(ch[u][0]);
    printf("%d ", key[u].first);
    if (ch[u][1]) debug(ch[u][1]);
  }
} splay;

char op[maxn][10];
int main() {
  int T, kase = 1;
  scanf("%d", &T);
  while (T--) {
    int n, q;
    scanf("%d%d", &n, &q);
    vector<int> num;
    for (int i = 0; i < q; i++) {
      scanf("%s%d", op[i], &query[i]);
      if (op[i][0] == 'R') continue;
      num.push_back(query[i]);
    }
    my_sort_unique(num);
    a.clear();
    int l = 1, r = 1e8 + 5;
    for (auto &v : num) {
      if (l < v) a.push_back({l, v - l});
      a.push_back({v, 1});
      l = v + 1;
    }
    a.push_back({l, r - l + 1});

    printf("Case %d:\n", kase++);
    splay.init();
    for (int i = 0; i < q; i++) {
      if (op[i][0] == 'T') {
        splay.Top(query[i]);
      } else if (op[i][0] == 'Q') {
        splay.query(query[i]);
      } else {
        splay.rank(query[i]);
      }
    }
  }
}
```