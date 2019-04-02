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
```

## 练习

**HDU2222: 一个主串，多个模式串，问主串中有多少种模式串**
记录单词结尾的val数组，访问一次之后就要清空，这样就可以统计种类了。

---

**HDU2896: 多个主串，多个模式串，问每个主串中有多少种模式串**
因为每个主串最多有3个模式串，所以可以记录一次匹配中有哪些val被访问了，匹配结束后恢复一下即可。

---