---
title: ACM模板
date: 2019-04-01 23:05:52
categories:
- ACM
tags:
- 模板
---

## 字符串
### KMP
```cpp
struct KMP{
    int back[maxn];
    void getfail(string line){
        int i,k;
        k = back[0] = -1;
        i = 0;
        while(i < line.size()){
            while(k != -1 && line[i] != line[k]) k = back[k];
            back[++i] = ++k;
        }
    }
    int match(string T,string P){
        int i = 0,k = 0;
        int ret = 0;
        getfail(P);
        while(i < T.size()){
            while(k != -1 && T[i] != P[k]) k = back[k];
            ++i;++k;
            if(k >= P.size()){
                ret++;
                k = back[k];
            }
        }
        return ret;
    }
};
```

### AC自动机
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
            int tmp = u;
            while (tmp != rt) {
                res += val[tmp];
                val[tmp] = 0;
                tmp = f[tmp];
            }
        }
        return res;
    }
}
```