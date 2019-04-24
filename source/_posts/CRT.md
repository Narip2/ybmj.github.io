---
title: CRT
date: 2019-04-24 10:21:23
categories:
  - ACM
  - 数论
tags:
  - 数论
  - 中国剩余定理
---

[参考博客](https://blog.csdn.net/niiick/article/details/80229217)

# 中国剩余定理

有一次同余方程组

$$
\begin{cases} x \equiv r_1\quad mod(m_1)\\\\
x \equiv r_2\quad mod(m_2)\\\\
x \equiv r_3\quad mod(m_3)\\\\
...
\end{cases}
$$

**条件：**$m_i$两两互质。

**解法：**

设$M=lcm(m_1,m_2,...)$。

设$t_i$满足$t_i \times \frac{M}{m_i} \equiv 1 \quad mod (m_i)$，该方程有解有且当且$m_i$与$\frac{M}{m_i}$互质（证明参考扩展欧几里得算法）。

那么有特解$x = \sum (r_i\times t_i\times \frac{M}{m_i})$。

通解为$x + k\times M (k \in Z)$

```cpp
// mod两两互质
// 通解 re + k * M
// 返回最小非负整数解
void crt(ll r[], ll m[], ll n, ll &re, ll &M)
{
    M = 1, re = 0;
    for (int i = 0; i < n; i++) M *= m[i];
    for (int i = 0; i < n; i++)
    {
        ll x, y,  tm = M / m[i];
        ll d = exgcd(tm, m[i], x, y);
        re = (re + tm * x * r[i]) % M;
    }
    re = (re + M) % M;
}
```

# 扩展中国剩余定理

**条件：**$m_i$不满足两两互质

**解法：**

假设已经求得前$k-1$个同余方程的通解$x + k\times M (k \in Z)$，其中
$M=\prod_{i=1}^{k-1}m_i$。

那么加入第$k$个方程$x \equiv a_k \quad mod(m_k)$后，新的解满足$x + t\times M \equiv a_k \quad mod(m_k)$，只要求出$t$即可。

求解上述方程可用扩展欧几里得。

若有一个同余方程无解，则整个一次同余方程组无解。

所以我们只要求$k$次扩展欧几里得即可。

```cpp
// mod不满足两两互质
// 通解为 re + k*M
// 返回最小非负整数解
bool excrt(ll r[], ll m[], ll n, ll &re, ll &M)
{
    ll x, y;
    M = m[0], re = r[0];
    for (int i = 1; i < n; i++)
    {
        ll d = exgcd(M, m[i],  x, y);
        if ((r[i] - re) % d != 0) return 0;
        x = (r[i] - re) / d * x % (m[i] / d);
        re += x * M;
        M = M / d * m[i];
        re %= M;
    }
    re = (re + M) % M;
    return 1;
}
```
