---
title: concrete-math-integers-floors-and-ceilings
date: 2024-04-09 10:02:06
tags:
---

# 底和顶 Floors and Ceilings

整值函数中的底函数和顶函数的定义如下

- $\lfloor x \rfloor$ 为小于或等于 $x$ 的最大整数；
- $\lceil x \rceil$ 为大于或等于 $x$ 的最小整数。

整值函数拥有以下性质

- $\lceil x \rceil - \lfloor x \rfloor = [x \in Z]$,
- $x-1 < \lfloor x \rfloor \leq x \leq \lceil x \rceil < x + 1$,
- $\lfloor -x \rfloor = -\lceil x \rceil$,
- $\lceil -x \rceil = -\lfloor x \rfloor$,
- $\lfloor x+n \rfloor = \lfloor x \rfloor + n, n \in Z$,

对于任意连续、递增的函数 $f(x)$，如果其满足 $x,f(x) \in Z$，那么
$$
\begin{align*}
\lfloor f(x) \rfloor &= \lfloor f(\lfloor x \rfloor) \rfloor; \\
\lceil f(x) \rceil &= \lceil f(\lceil x \rceil) \rceil. \\
\end{align*}
$$
我们证明第二个式子，第一个式子同理可证。

如果 $x = \lceil x \rceil$，式子显然成立。如果 $x < \lceil x \rceil$，由于 $f(x)$ 递增，则有 $f(x)<f(\lceil x \rceil)$。不等式两边同时取整，有 $\lceil f(x) \rceil \leq \lceil f(\lceil x \rceil) \rceil$。要证明等式 $\lceil f(x) \rceil = \lceil f(\lceil x \rceil) \rceil$ 成立，则只需要证明不等式 $\lceil f(x) \rceil < \lceil f(\lceil x \rceil) \rceil$ 不成立即可。

假设 $\lceil f(x) \rceil < \lceil f(\lceil x \rceil) \rceil$ 成立，根据中值定理，存在 $x \leq y < \lceil x \rceil$，使得 $f(y) = \lceil f(x) \rceil$。但由于 $y$ 是一个整数，不存在一个整数位于 $\lfloor x \rfloor$ 与 $\lceil x \rceil$ 之间，与命题矛盾，假设不成立。所以等式成立。
