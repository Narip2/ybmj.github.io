---
title: 2018 ICPC Asia Singapore Regional Contest
date: 2019-10-08 17:29:00
categories:
- ACM
- 题目
tags:
- ICPC
---

# F - Wi Know 


## 题意
给定一个长度为 $n$ 的序列，找一个字典序最小的子序列 `[A,B,A,B]`，要求 $A$ 不等于 $B$。若没有找到输出 $-1$。

$1 \leq n \leq 4e5, 1 \leq a[i] \leq n$

## 分析

从左到右枚举 $a[i]$ 为第一个 $B$。记下一个 $B$ 出现的位置为 $p$。那么我们需要在 $(i,p)$ 这个区间内找到一个值最小的 $A$（线段树维护）。 问题是如何知道这个 $A$ 在 $[1,i)$ 是否出现过？我们只要保证任何时刻在线段树中的元素都是合法的即可。即枚举完当前元素后，将其下一次出现的位置的值更新为 $a[i]$。这样每次只要在线段树中查询最小值即可，如果查到正无穷表示不合法。 

## 代码
```cpp
#include<bits/stdc++.h>
using namespace std;
#define lson (rt << 1)
#define rson (rt << 1 | 1)
#define Lson l, m , lson
#define Rson m + 1, r , rson

const int maxn = 4e5+5;
int a[maxn], nxt[maxn], pos[maxn];
const int INF = 0x3f3f3f3f;
int seg[maxn << 2];

void update(int l,int r,int rt,int p,int x){
    if(l == r) {
        seg[rt] = x;
        return ;
    }
    int m = l + r >> 1;
    if(p <= m) update(Lson, p, x);
    else update(Rson, p, x);
    seg[rt] = min(seg[lson], seg[rson]);
}
int query(int l,int r,int rt,int L,int R){
    if(L > R) return INF;
    if(L <= l && R >= r) return seg[rt];
    int m = l + r >>1 , ret = INF;
    if(L <= m) ret = min(ret, query(Lson,L,R));
    if(m+1 <= R) ret = min(ret, query(Rson, L,R));
    return ret;
}

int main(){
    int n;
    scanf("%d",&n);
    for(int i=1;i<=n;i++) scanf("%d",a+i), pos[i] = -1;
    for(int i=n;i>0;i--){
        nxt[i] = pos[a[i]];
        pos[a[i]] = i;
    }
    memset(seg,0x3f,sizeof(seg));
    pair<int,int> ans = {INF,INF};
    for(int i=1;i<=n;i++){
        if(nxt[i] != -1){
            int tmp = query(1,n,1,i+1,nxt[i]-1);
            if(tmp != INF)
                ans = min(ans, {tmp, a[i]});
            update(1,n,1,nxt[i],a[i]);
        }
    }
    if(ans.first == INF) puts("-1");
    else printf("%d %d\n",ans.first, ans.second);
}
```


# G - Rectangular City

## 题意
一个 $R \times C$，要求依次选 $n$ 个矩形（可以相同），使得这些矩形的交的面积不小于 $k$ 。求方案数，结果取模。

$1 \leq R,C \leq 5000$
## 分析

选矩形的操作可以简化为在两个一维空间中分别选一段区间，其区间长度乘积不小于 $k$。


设 $dp[len]$ 表示在长度为 $r$ 的一维空间中，选 $n$ 个区间，使得这些区间交的长度为 $len$。

那么只要通过枚举起点即可计算 $dp[len]$。

公式为：若区间左边有 $a$ 个位置可以选，右边有 $b$ 个元素可以选，那么 $dp[len] += (a^n - (a-1)^n) \times (b^n - (b-1)^n)$。

最后只要枚举行和列各自所选的长度求和即可。


## 代码
```cpp
#include <bits/stdc++.h>
using namespace std;
const int mod = 1e9 + 7;
const int maxn = 5e3 + 5;
int dp[2][maxn];
int npow[maxn];

int Pow(int a, int b) {
  int ret = 1;
  while (b) {
    if (b & 1) ret = 1LL * ret * a % mod;
    a = 1LL * a * a % mod;
    b >>= 1;
  }
  return ret;
}
inline int cal(int x) { return (npow[x] - npow[x - 1] + mod) % mod; }
int main() {
  int n, sz[2], k;
  cin >> n >> sz[0] >> sz[1] >> k;
  for (int i = 0; i < maxn; i++) npow[i] = Pow(i, n);

  for (int i = 0; i < 2; i++) {
    for (int len = 1; len <= sz[i]; len++) {
      for (int j = 0; j <= sz[i] - len; j++) {
        int a = j + 1, b = sz[i] - j - len + 1;
        int tmp = 1LL * cal(a) * cal(b) % mod;
        dp[i][len] = (dp[i][len] + tmp) % mod;
      }
    }
  }
  int ans = 0;
  for (int i = 1; i <= sz[0]; i++) {
    for (int j = 1; j <= sz[1]; j++) {
      if (i * j >= k) ans = (ans + 1LL * dp[0][i] * dp[1][j] % mod) % mod;
    }
  }
  cout << ans << endl;
}
```