---
title: hihoCoder1879 Rikka with Triangles
date: 2019-10-31 20:52:00
categories:
- ACM
- 题目
tags:
- 极角排序
---

## 题意

平面上有 $n$ 个点，求锐角三角形的总面积。存在三点共线。

$1 \leq n \leq 2000$

## 分析

直接求锐角并不好求，考虑用总面积减去直角三角形和钝角三角形的面积。

我们可以通过枚举顶点，然后极角排序。计算顶点这个角是锐角三角形的总面积和是直角和钝角的总面积。这里维护一个滑动窗口即可（因为三点共线的情况，这里的细节较多。推荐先对同方向的向量进行合并）。

然后我们发现对于一个锐角三角形，我们会算三遍它的面积。对于对于一个直角或钝角三角形，它们会被当做锐角三角形算两次，因此我们要减去二倍的直角或钝角三角形的面积。

最后的答案再除以三即可。

## 代码
```cpp
#include <bits/stdc++.h>
using namespace std;
const int mod = 998244353;
using ll = long long;
using i128 = __int128;

const int maxn = 2e3 + 5;

inline int sign(i128 a) { return a < 0 ? -1 : a > 0; }
inline int cmp(i128 a, i128 b) { return sign(a - b); }
struct Point {
  i128 x, y;

  Point operator+(const Point &rhs) const { return {rhs.x + x, rhs.y + y}; }
  Point operator-(const Point &rhs) const { return {x - rhs.x, y - rhs.y}; }
  Point operator*(i128 rhs) const { return {x * rhs, y * rhs}; }
  Point operator/(i128 rhs) const { return {x / rhs, y / rhs}; }

  bool operator<(const Point &rhs) const {
    int s = cmp(x, rhs.x);
    if (s) return s == -1;
    return cmp(y, rhs.y) == -1;
  }
  bool operator==(const Point &rhs) const {
    return cmp(x, rhs.x) == 0 && cmp(y, rhs.y) == 0;
  }

  void Mod() {
    x %= mod;
    y %= mod;
  }

  // 是否在上半区
  bool quad() const { return sign(y) == 1 || (sign(y) == 0 && sign(x) >= 0); }

  i128 dot(Point rhs) { return x * rhs.x + y * rhs.y; }
  i128 det(Point rhs) { return x * rhs.y - y * rhs.x; }
};
bool cmpAngle(Point a, Point b) {
  if (a.quad() != b.quad())
    return a.quad() < b.quad();
  else
    return sign(a.det(b)) > 0;
}

Point a[maxn], ps[maxn], pp[maxn];
ll inv3 = 332748118;

ll solve(int n) {
  if (n <= 1) return 0;
  Point pt1 = pp[1], pt2 = pp[1];
  ll ret = 0;
  int cur1 = 1, cur2 = 1;
  for (int i = 1; i <= n; i++) {
    while (true) {
      int nxt = cur1 + 1;
      if (nxt > n) nxt = 1;
      if (nxt == i) break;
      if (!(ps[i].det(ps[nxt]) >= 0 && ps[i].dot(ps[nxt]) > 0)) break;
      pt1 = pt1 + pp[nxt];
      cur1 = nxt;
    }
    while (true) {
      int nxt = cur2 + 1;
      if (nxt > n) nxt = 1;
      if (nxt == i) break;
      if (!(ps[i].det(ps[nxt]) >= 0)) break;
      pt2 = pt2 + pp[nxt];
      cur2 = nxt;
    }

    auto tmp1 = pt1, tmp2 = pt2 - pt1;
    tmp1.Mod(), tmp2.Mod();
    ret = (ret + pp[i].det(tmp1)) % mod;
    ret = (ret - pp[i].det(tmp2) * 2LL % mod + mod) % mod;

    if (cur1 == i) {
      cur1++;
      pt1 = pt1 + pp[cur1];
    }
    if (cur2 == i) {
      cur2++;
      pt2 = pt2 + pp[cur2];
    }
    pt1 = pt1 - pp[i];
    pt2 = pt2 - pp[i];
  }
  return ret;
}
int main() {
  int T, n;
  scanf("%d", &T);
  while (T--) {
    scanf("%d", &n);
    ll x, y;
    for (int i = 1; i <= n; i++) scanf("%lld%lld", &x, &y), a[i] = {x, y};

    ll tot = 0;
    for (int i = 1; i <= n; i++) {
      int sz = 0, m = 1;
      for (int j = 1; j <= n; j++)
        if (j != i) ps[++sz] = a[j] - a[i];

      sort(ps + 1, ps + 1 + sz, cmpAngle);
      pp[1] = ps[1];
      for (int i = 2; i <= sz; i++) {
        if (ps[m].det(ps[i]) == 0 && ps[m].dot(ps[i]) > 0) {
          pp[m] = pp[m] + ps[i];
        } else {
          ps[++m] = ps[i];
          pp[m] = ps[i];
        }
      }
      if (m == 1) continue;
      if (ps[m].det(ps[1]) == 0 && ps[m].dot(ps[1]) > 0) {
        pp[1] = pp[1] + ps[m];
        m--;
      }

      tot = (tot + solve(m)) % mod;
    }
    printf("%lld\n", tot * inv3 % mod);
  }
}
```
