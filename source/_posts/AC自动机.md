---
title: AC自动机
date: 2019-04-02 10:06:57
categories:
- ACM
tags:
- 字符串
- AC自动机
---

## 介绍

[参考博客1](https://www.luogu.org/blog/42196/qiang-shi-tu-xie-ac-zi-dong-ji)
[参考博客1](http://www.cnblogs.com/cjyyb/p/7196308.html)

## 复杂度分析
时间复杂度：
- `insert`：主串的长度
- `build`：结点数×字符集大小
- `query`：主串的长度？

空间复杂度：
- 结点数 约等于 模式串的大小×模式串的数量
<!--more-->
## 模板
```cpp
struct Trie {
    int ch[maxn][26], f[maxn], val[maxn];
    int sz, rt;
    int newnode() {
        memset(ch[sz], -1, sizeof(ch[sz])), val[sz] = 0;
        return sz++;
    }
    void init() { sz = 0, rt = newnode(); }
    int idx(char c) { return c - 'A'; }
    void insert(const char* s) {
        int u = 0;
        for (int i = 0; i < s[i]; i++) {
            int c = idx(s[i]);
            if (ch[u][c] == -1) ch[u][c] = newnode();
            u = ch[u][c];
        }
        val[u]++;
    }
    void build() {
        queue<int> q;
        f[rt] = rt;
        for (int c = 0; c < 26; c++) {
            if (~ch[rt][c])
                f[ch[rt][c]] = rt, q.push(ch[rt][c]);
            else
                ch[rt][c] = rt;
        }
        while (!q.empty()) {
            int u = q.front();
            q.pop();
            for (int c = 0; c < 26; c++) {
                if (~ch[u][c])
                    f[ch[u][c]] = ch[f[u]][c], q.push(ch[u][c]);
                else
                    ch[u][c] = ch[f[u]][c];
            }
        }
    }
    int query(const char* s) {
        int u = rt;
        int res = 0;
        for (int i = 0; s[i]; i++) {
            int c = idx(s[i]);
            u = ch[u][c];
            for(int j=u;j!=rt&&~val[j];j=f[j])
                res+=val[j],val[j]=-1;
        }
        return res;
    }
}
```cpp
// 指针版
struct Node {
    Node *f, *ch[Types];
    int val;
    Node() {
        val = 0;
        f = nullptr;
        for (int i = 0; i < Types; i++) ch[i] = nullptr;
    }
};
struct Trie {
    vector<Node *> nodes;  // 全部结点，方便删除
    Node *rt;
    void init() {
        rt = new Node();
        nodes.push_back(rt);
    }
    int idx(const char &c) { return c - 'A'; }
    void insert(int id, const char *s) {
        Node *u = rt;
        for (int i = 0, c; s[i]; i++) {
            c = idx(s[i]);
            if (u->ch[c] == nullptr)
                u->ch[c] = new Node(), nodes.push_back(u->ch[c]);
            u = u->ch[c];
        }
        u->val++;
    }
    void build() {
        queue<Node *> q;
        rt->f = rt;
        for (int i = 0; i < Types; i++) {
            if (rt->ch[i] != nullptr)
                rt->ch[i]->f = rt, q.push(rt->ch[i]);
            else
                rt->ch[i] = rt;
        }
        while (!q.empty()) {
            Node *u = q.front();
            q.pop();
            for (int i = 0; i < Types; i++) {
                if (u->ch[i] != nullptr)
                    u->ch[i]->f = u->f->ch[i], q.push(u->ch[i]);
                else
                    u->ch[i] = u->f->ch[i];
            }
        }
    }
    int query(const char *s) {
        int ret = 0;
        Node *u = rt;
        for (int i = 0, c; s[i]; i++) {
            c = idx(s[i]);
            u = u->ch[c];
            for (auto p = u; p != rt && p->val != -1; p = p->f) {
                ret += p->val;
                p->val = -1;
            }
        }
        return ret;
    }
    void delTrie() {
        for (auto &v : nodes) delete v;
        nodes.clear();
    }
} trie;
```

## 练习

**HDU2222: 一个主串，多个模式串，问主串中有多少种模式串**
记录单词结尾的val数组，访问一次之后就要清空，这样就可以统计种类了。

---

**HDU2896: 多个主串，多个模式串，问每个主串中有多少种模式串**
因为每个主串最多有3个模式串，所以可以记录一次匹配中有哪些val被访问了，匹配结束后恢复一下即可。

---

**ZOJ3494: 问[A,B]内有多少数字，其bcd表示不含模式串**
数位dp的状态用ac自动机上的结点来表示。每加一位数字，相当于在ac自动机上转移四次(BCD码将一位数字表示成四个二进制位)。优化时间：用val数组表示是否会出现模式串，在build的时候如果val[f[u]]大于0，那么说明val[u]也要大于0.这样防止查询的时候在自动机上来回跳。

---

**POJ2778: 问有多少个长度为n的串不包含模式串，n为2e9，模式串总长度小于100**
通过ac自动机构造可达矩阵(结点i到结点j的方案数)，可达矩阵的n次幂，就是i到j有多少条长度为n的路径。最后统计$\sum_{i=0}^{sz}mat[0][i]$即可

---

**HDU2243: 问有多少个长度至多为n的串不包含模式串**
相对于上一题多了一个求和，需要用到公式：

$$
 \left(
 \begin{matrix}
   A & E \\\\
   0 & E 
  \end{matrix}
  \right)^n = 
 \left(
 \begin{matrix}
   A^n & E+A+A^2+...+A^{n-1} \\\\
   0 & E 
  \end{matrix}
  \right)
$$
---