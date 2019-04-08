---
title: RSA加密算法
date: 2019-04-08 15:17:48
categories:
- 加密算法
tags:
- rsa
---

# RSA非对称加密通信过程

我拥有私钥，我将公钥交给想和我通信的人。

他发送消息时用公钥加密，我收到消息后用私钥解密。

我发送消息时用私钥加密，他收到消息后用公钥解密。

# 算法原理

## 密钥生成

1. 选两个质数p和q
2. $n = pq$
3. $\phi(n) = (p-1)\times (q-1)$
4. $(n,e)$构成公钥，$1 < e < \phi(n)$ 且 $e$与$\phi(n)$互质
5. $(n,d)$构成私钥，$ed \equiv 1 \quad mod \quad \phi(n)$

## 加密过程

消息$m$

1. 公钥(私钥)加密： $c = m^e \quad mod \quad n$
2. 私钥(公钥)解密： $m = c^d \quad mod \quad n$

## 原理

以公钥加密，私钥解密为例：

我们需要验证：$m \equiv m^{ed} \quad mod \quad n$

因为 $ed \equiv 1 \quad mod \quad \phi(n)$

所以 $ed = 1 + k\phi(n)$

-> $m^{ed} = m^{1 + k\phi(n)}$

**当n与m互质时：**

由欧拉定理可知：$m^{\phi(n)} \equiv 1 \quad mod \quad n$

因此：$m^{1 + k\phi(n)} \equiv m \quad mod \quad n$

即：$m \equiv m^{ed} \quad mod \quad n$

**当n与m不互质时：**

因为$n=pq$，因此$m=kp$或$m=kq$，不妨设$m=kp$

$m^{\phi(q)} = (kp)^{\phi(q)} \equiv 1 \quad mod \quad q$

-> $(kp)^{h\phi(p) \phi(q) + 1} \equiv (kp) \quad mod \quad q$

-> $(kp)^{ed} \equiv (kp) \quad mod \quad q$

-> $(kp)^{ed} = kp + tq$

因为$kp + tq \equiv 0 \quad mod \quad p$

因此$t = rp$

-> $(kp)^{ed} = kp + rpq$

-> $m^{ed} \equiv m \quad mod \quad n$

证毕