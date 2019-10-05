---
title: cf1234F Yet Another Substring Reverse
date: 2019-10-05 21:41:17
categories:
- ACM
- 题目
tags:
- 子集dp
---

[题目链接](https://codeforces.com/contest/1234/problem/F)

# 题目
给一个长度为 $n$ 的只包含 $20$ 个小写字母的字符串，现在最多可进行一次区间反转。问最长的子串的长度？ 要求这个子串中所有字母都是不同的。

$1 \leq n \leq 10^6$

# 分析

一直在考虑会不会有重叠的问题，后来发现不会出现重叠的情况。

$dp[i]$ 表示 $i$ 所表示的二进制中，那些位为 $1$ 的元素 最多有多少个存在。

转移可以通过枚举所有为 $1$ 的位，来进行转移。


# 代码

```cpp
#include <bits/stdc++.h>
using namespace std;
const int maxn = (1 << 20) + 5;
bool bit[maxn];
int dp[maxn];

int main() {
  string s;
  cin >> s;
  for (int i = 0; i < s.size(); i++) {
    int mask = 0;
    for (int j = i; j < s.size(); j++) {
      if (mask & (1 << (s[j] - 'a'))) break;
      mask |= 1 << (s[j] - 'a');
      bit[mask] = 1;
    }
  }
  for (int i = 0; i < (1 << 20); i++)
    if (bit[i]) dp[i] = __builtin_popcount(i);
  for (int i = 0; i < (1 << 20); i++) {
    for (int j = 0; j < 20; j++) {
      if (i & (1 << j)) dp[i] = max(dp[i], dp[i ^ (1 << j)]);
    }
  }
  int ans = 0;
  for (int i = 0; i < (1 << 20); i++)
    if (bit[i]) ans = max(ans, __builtin_popcount(i) + dp[i ^ ((1 << 20) - 1)]);
  cout << ans << endl;
}
```
