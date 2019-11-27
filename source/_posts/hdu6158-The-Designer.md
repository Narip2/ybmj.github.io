---
title: hdu6158 The Designer
date: 2019-11-28 00:22:09
categories:
- ACM
- 题目
tags:
- 计算几何
- 反演变换
---

[题目链接](http://acm.hdu.edu.cn/showproblem.php?pid=6158)

## 题意

有点难描述，还是直接阅读原题吧。

## 分析

设两圆切点为 $O$。可以默认其为坐标原点。两个圆反演之后就是两条直线。


![](https://ybmj-blog-1256173108.cos.ap-shanghai.myqcloud.com/blog-picture/hdu6158.png)

[图片来源](https://www.cnblogs.com/NineSwords/p/9225187.html)


然后接下来只要计算每个小圆的半径即可。

对于 $N$ 很大的情况，这里的做法是当每个小圆的面积趋近于 $eps$ 时就停止计算。

## 代码
```cpp
#include <bits/stdc++.h>
using namespace std;
using db = double;
const db eps = 1e-14;
const db PI = acos(-1.0);
inline int sign(db a) { return a < -eps ? -1 : a > eps; }
inline int cmp(db a, db b) { return sign(a - b); }

struct Point {
  db x, y;
  Point(db x = 0, db y = 0) : x(x), y(y) {}

  bool operator<(const Point& rhs) const {
    int s = cmp(x, rhs.x);
    if (s) return s == -1;
    return cmp(y, rhs.y) == -1;
  }
  bool operator==(const Point& rhs) const {
    return cmp(x, rhs.x) == 0 && cmp(y, rhs.y) == 0;
  }

  void print() { cerr << "Point : " << x << ' ' << y << endl; }
};
typedef Point Vector;

Vector operator+(Vector A, Vector B) { return Vector(A.x + B.x, A.y + B.y); }
Vector operator-(Vector A, Vector B) { return Vector(A.x - B.x, A.y - B.y); }
Vector operator*(Vector A, db p) { return Vector(A.x * p, A.y * p); }
Vector operator/(Vector A, db p) { return Vector(A.x / p, A.y / p); }

db Dot(Vector A, Vector B) { return A.x * B.x + A.y * B.y; }
db Length(Vector A) { return sqrt(Dot(A, A)); }
db Length2(Vector A) { return Dot(A, A); }
db Cross(Vector A, Vector B) { return A.x * B.y - A.y * B.x; }

struct Line {
  Point u;
  Vector v;
  Line() {}
  Line(Point u, Vector v) : u(u), v(v) {}
};

struct Circle {
  Point c;
  db r;
  Circle() : c(Point(0, 0)), r(0) {}
  Circle(Point c, db r = 0) : c(c), r(r) {}

  // 根据角度a，返回圆上一点
  Point point(db a) { return Point(c.x + cos(a) * r, c.y + sin(a) * r); }
};

// 经过a点，与向量v垂直的垂线
Line Perpendicular(Point a, Vector v) {
  if (sign(v.y) == 0)
    return {a, Point(0, 1)};
  else {
    Point u = Point(0, v.x * a.x / v.y + a.y);
    return {a, u - a};
  }
}

// 过O的圆，反演后得到的直线
Line Inversion_C2L(Point O, db R, Circle A) {
  Line line = Line(O, A.c - O);
  db rate = R * R / (2 * A.r);
  Point b = line.v * (rate / Length(line.v)) + O;
  return Perpendicular(b, A.c - O);
}

int main() {
  int T;
  scanf("%d", &T);
  while (T--) {
    db r1, r2;
    int n;
    scanf("%lf%lf%d", &r1, &r2, &n);
    if (r1 > r2) swap(r1, r2);
    db R = 3 * r2 + 5;
    Circle O(Point(0, 0), R);
    Circle A(Point(r1, 0), r1), B(Point(r2, 0), r2);
    Line left = Inversion_C2L(O.c, R, B);
    Line right = Inversion_C2L(O.c, R, A);
    db r = (right.u.x - left.u.x) / 2;
    db mid = (left.u.x + right.u.x) / 2;
    db a = mid * mid, b = 0;
    db rr = R * R * r / (a - r * r);
    db ans = PI * rr * rr;
    for (int i = 2; i <= n; i++) {
      if (!(i & 1)) {
        b += 2 * r;
        a = mid * mid + b * b;
        rr = R * R * r / (a - r * r);
      }
      db area = PI * rr * rr;
      ans += area;
      if (sign(area) < 1) break;
    }
    printf("%.5f\n", ans);
  }
}
```