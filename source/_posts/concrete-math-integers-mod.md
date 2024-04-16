---
title: concrete-math-integers-mod
date: 2024-04-14 10:39:16
tags:
---

# 模运算 Mod

当 $m,n$ 是正整数时，$n$ 被 $m$ 除的商是 $\lfloor n/m \rfloor$，余数记为 $n \bmod m$，称作 $n$ 对 $m$ 取模。取模运算可以扩展到实数域，$x \bmod y$ 的物理意义是，当 $x$ 和 $y$ 是正实数时，想象一个周长为 $y$ 的圆，它的点被赋予区间 $[0..y)$ 中的实数．如果我们从 $0$ 出发，绕着圆行走距离 $x$ ，我们就停止在 $x \bmod y$ 的位置。
$$
x \bmod y = x - y \lfloor x/y \rfloor, y \neq 0.
$$
以下是取模运算的一些性质。
$$
x \bmod 0 = x; \\
x = \lfloor x \rfloor + x \bmod 1; \\
c(x \bmod y) = (cx) \bmod (cy);\\
(x+y) \bmod p = (x \bmod p + y \bmod p) \bmod p
$$
现在我们考虑一个问题，如何将 $n$ 件物品平均分配给 $m$ 个人？

很容易想到，前 $n \bmod m$ 个人分配 $\lceil n/m \rceil$ 件物品，剩下的人分配 $\lfloor n/m \rfloor$ 件物品。使用统一的方法表示每个人分配到的物品数量就是
$$
\lceil \frac{n-k+1}{m} \rceil, 1 \leq k \leq m.
$$
假设 $n=qm+r, 0 \leq r < m$，那么
$$
\lceil \frac{n-k+1}{m} \rceil = \lceil \frac{qm+r-k+1}{m} \rceil = q+\lceil \frac{r-k+1}{m} \rceil.
$$
当 $1 \leq k \leq r$ 时，$\lceil \frac{r-k+1}{m} \rceil=1$；当 $r < k \leq m$ 时，$\lceil \frac{r-k+1}{m} \rceil=0$。因此
$$
n = \lceil \frac{n}{m} \rceil + \lceil \frac{n+1}{m} \rceil + \cdots + \lceil \frac{n+m-1}{m} \rceil.
$$
由 $n= \lfloor \frac{n}{2} \rfloor + \lceil \frac{n}{2} \rceil$ 可以将上式转换为
$$
n = \lfloor \frac{n}{m} \rfloor + \lfloor \frac{n+1}{m} \rfloor + \cdots + \lfloor \frac{n+m-1}{m} \rfloor.
$$
令 $n= \lfloor mx \rfloor$，则有
$$
\lfloor mx \rfloor = \lfloor x \rfloor + \lfloor x+\frac{1}{m} \rfloor + \cdots + \lfloor x + \frac{m-1}{m} \rfloor.
$$
