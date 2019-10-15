---
title: 2019 German Collegiate Programming Contest (GCPC 19)
date: 2019-10-14 11:04:42
categories:
- ACM
- 题目
tags:
- GCPC
---

# K - Keeping the Dogs Out

## 题意
给定若干边长为 $2$ 的幂次的小正方形，问是否能拼成一个矩形，如果可以的话输出其长和宽。保证矩形面积小于 $10^15$。

## 分析

因为保证了矩形面积，因此可以枚举长和宽（因子个数的数量级为 $\log$）。

设枚举的长和宽分别为 $a,b (a < b)$。

设所有边长 $\geq 2^i$ 的小正方形的面积和 $sum_i$。

如果矩形只用 $\geq 2^i$ 的小正方形，那么其面积为 $area_i = \lfloor \frac{a}{2^i} \rfloor \times 2^i \times \lfloor \frac{b}{2^i} \rfloor \times 2^i$

如果 $a,b$ 合法，那么对于每个 $i$，必须有 $sum_i \leq area_i$。（如果 $sum_i < area_i$，那么空余的面积可以用边长小于 $2^i$ 的小正方形来填充。）


## 代码
```cpp
#include <bits/stdc++.h>
using namespace std;
using ll = long long;
ll m[30], suf[30], pow2[30];
int main() {
  int n;
  cin >> n;
  ll area = 0, tmp = 1;
  for (int i = 0; i <= n; i++) cin >> m[i], area += m[i] * tmp, tmp *= 4;
  for (int i = n; i >= 0; i--) tmp /= 4, suf[i] = suf[i + 1] + m[i] * tmp;
  pow2[0] = 1;
  for (int i = 1; i <= n; i++) pow2[i] = pow2[i - 1] * 2;

  ll sqa = sqrt(area);
  for (ll i = 1; i <= sqa; i++) {
    if (area % i) continue;
    ll a = i, b = area / i;
    bool ok = true;
    for (int j = 0; j <= n; j++) {
      if (!m[j]) continue;
      if (a < pow2[j] || b < pow2[j]) {
        ok = false;
        break;
      }
      ll tmpa = a / pow2[j] * pow2[j], tmpb = b / pow2[j] * pow2[j];
      if (tmpa * tmpb < suf[j]) {
        ok = false;
        break;
      }
    }
    if (ok) {
      cout << a << ' ' << b << endl;
      return 0;
    }
  }
  cout << "impossible" << endl;
}
```