---
title: 2018 ICPC Mid-Central Regional
date: 2019-10-23 19:35:54
categories:
- ACM
- 题目
tags:
- ICPC
---

# I - Random Manhattan Distance

## 题意

给定 $n$ 个点的凸包，问从凸包内任选两点，其曼哈顿距离的期望是多少。

$3 \leq n \leq 100000$

## 分析

首先两维是独立的，对于任意一维来说：

设凸包面积为 $S$。

设 $weight(x) = leftHeight + x \times \frac{(rightHeight - leftHeight)}{width}$，表示所选点落在 $x$ 位置的概率乘 $S$（这里是为了减小误差，最后再除掉即可。先当作概率来理解）。

因为这个概率密度函数 $weight(x)$ 是个分段函数，每一段对应一个梯形（扫描线从左到右扫过去）。因此我们将答案分为两部分来算，第一部分是当前梯形所对应的区间内任选两点的贡献；第二部分是当前梯形所对应的区间内选一点，之前计算过的区间内选一点，这两个点的贡献。

第一部分的答案就是 $2\times \int_{x=0}^{w} \int_{y=0}^{x} weight(x) * weight(y) * (x - y) dy\ dx$

> 积分全靠 $wolframalpha$...

第二部分的答案是当前梯形所对应区间的期望位置和之前区域的期望位置之差。

当前期望位置的计算：$current_x + E(x) =current_x + \int_0^w x \times (\frac{weight(x)}{area})$

之前区域期望位置的计算：$\frac{E_0 \times area_0+ E_1 \times area_1 + ...}{area_0 + area_1 + ...}$

> 期望计算公式：$E(g(X)) = \int g(x)f(x) dx$

## 代码
```cpp
#include <bits/stdc++.h>
using namespace std;

const int MAXN = 100000;
const long long INF = 1000000000;

struct Point {
  long long x, y;

  Point() {}
  Point(long long x, long long y) : x(x), y(y) {}
};

bool operator<(const Point& a, const Point& b) {
  return a.x < b.x || a.x == b.x && a.y < b.y;
}

Point operator-(const Point& a, const Point& b) {
  return Point(a.x - b.x, a.y - b.y);
}

long long det(const Point& a, const Point& b) { return a.x * b.y - a.y * b.x; }

struct Zone {
  double width, leftHeight, rightHeight;
  Zone() {}
  Zone(double width, double leftHeight, double rightHeight)
      : width(width), leftHeight(leftHeight), rightHeight(rightHeight) {}
};

double getHeight(const vector<Point>& a, long long x, int i, int j) {
  if (a[i].x == x) {
    return a[i].y;
  }
  if (a[j].x == x) {
    return a[j].y;
  }
  double slope = (a[j].y - a[i].y) / (double)(a[j].x - a[i].x);
  return a[i].y + slope * (x - a[i].x);
}

double solve(vector<Point>& a) {
  int n = a.size();
  for (int i = 0; i < n; ++i) {
    const Point& x = a[i];
    const Point& y = a[(i + 1) % n];
    const Point& z = a[(i + 2) % n];
    assert(det(z - y, y - x) > 0);
  }
  int upper_ptr =
      min_element(a.begin(), a.end()) - a.begin();  // in the +1 direction
  int lower_ptr = upper_ptr;                        // in the -1 direction
  vector<Zone> zones;
  while (true) {
    int upper_next = (upper_ptr + 1) % n;
    int lower_next = (lower_ptr - 1 + n) % n;
    long long current_x = max(a[upper_ptr].x, a[lower_ptr].x);
    if (current_x == a[upper_next].x) {
      upper_ptr = upper_next;
      continue;
    }
    if (current_x == a[lower_next].x) {
      lower_ptr = lower_next;
      continue;
    }
    long long next_x = min(a[upper_next].x, a[lower_next].x);
    if (next_x < current_x) {
      break;
    }
    double leftHeight = getHeight(a, current_x, upper_ptr, upper_next) -
                        getHeight(a, current_x, lower_ptr, lower_next);
    double rightHeight = getHeight(a, next_x, upper_ptr, upper_next) -
                         getHeight(a, next_x, lower_ptr, lower_next);
    zones.push_back(Zone(next_x - current_x, leftHeight, rightHeight));
    // fprintf(stderr, "%lld -> %lld\n", current_x, next_x);
    if (next_x == a[upper_next].x) {
      upper_ptr = upper_next;
    }
    if (next_x == a[lower_next].x) {
      lower_ptr = lower_next;
    }
  }
  double s = 0, a2 = 0, current_x = 0;
  double sum_area_x = 0, sum_area = 0;
  for (Zone zone : zones) {
    // Within the zone
    //      x from 0..width
    //      weight(x) = leftHeight + x * (rightHeight - leftHeight) / width
    //      \int_{x=0}^{w} \int_{y=0}^{x} weight(x) * weight(y) * (x - y) dx, dy
    s += (zone.leftHeight * zone.leftHeight +
          3 * zone.leftHeight * zone.rightHeight +
          zone.rightHeight * zone.rightHeight) *
         zone.width * zone.width * zone.width / 15;
    double area = (zone.leftHeight + zone.rightHeight) * zone.width / 2;
    // E[x] with in this zone
    double expected_x =
        current_x + (zone.leftHeight + 2 * zone.rightHeight) /
                        (3 * (zone.leftHeight + zone.rightHeight)) * zone.width;
    if (sum_area > 0) {
      // for the left zones
      s += 2 * (expected_x - sum_area_x / sum_area) * sum_area * area;
    }
    a2 += area * (2 * sum_area + area);  // S^2
    sum_area += area;
    sum_area_x += expected_x * area;
    current_x += zone.width;
  }
  return s / a2;
}

int main() {
  int n;
  assert(scanf("%d", &n) == 1);
  assert(3 <= n && n <= MAXN);
  vector<Point> a(n);
  for (int i = 0; i < n; ++i) {
    assert(scanf("%lld%lld", &a[i].x, &a[i].y) == 2);
    assert(abs(a[i].x) <= INF);
    assert(abs(a[i].y) <= INF);
  }
  double ret = solve(a);
  for (int i = 0; i < a.size(); ++i) {
    swap(a[i].x, a[i].y);
  }
  reverse(a.begin(), a.end());  // make sure it's in the counter-clockwise order
  ret += solve(a);
  printf("%.10f\n", ret);
  return 0;
}
```
