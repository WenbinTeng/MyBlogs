---
title: concrete-math-numbers-Mobius-inversion
date: 2024-04-21 08:42:52
tags:
---

# 莫比乌斯反演 Mobius Inversion Formula

对于一些函数 $f(n)$，如果很难直接求出它的值，而容易求出其倍数和或约数和 $g(n)$，那么可以通过莫比乌斯反演简化运算，求得 $f(n)$ 的值。

莫比乌斯函数由如下等式定义
$$
\mu(m) = \left\{\begin{matrix}\begin{align*}
1, \quad &m=1; \\
(-1)^k, \quad &m=\prod_{i=1}^k p_i \and \gcd(p_{i_p},p_{i_q})=1; \\
0, \quad &others.
\end{align*}\end{matrix}\right.
$$
假设 $m$ 存在素因数分解 $m = p_1^{c_1} p_2^{c_2} \dots p_k^{c_k}$，莫比乌斯函数的意义为

- 当 $m=1$ 时，$\mu(m)=1$；
- 当 $m \neq 1$ 时，
  - 当存在 $i \in [1,k]$，使得 $c_i>1$ 时，$\mu(m)=0$，也就是说只要某个质因子出现的次数超过一次，$\mu(m)=0$。
  - 当任意 $i \in [1,k]$，使得 $c_i=1$ 时，$\mu(m)=(-1)^k$，也就是说每个质因子都仅仅只出现过一次时，$\mu(m)=(-1)^k$，此处 $k$ 指的便是仅仅只出现过一次的质因子的总个数。

莫比乌斯函数具有如下性质
$$
\sum_{d \mid m} \mu(m) = [m=1].
$$
其中 $\sum_{d \mid m}$ 表示将 $m$ 的所有因子 $d$ 代入求和。莫比乌斯函数的前几项的值如下所示

| $m$      | 1    | 2    | 3    | 4    | 5    | 6    | 7    | 8    | 9    | 10   | 11   | 12   |
| -------- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| $\mu(m)$ | 1    | -1   | -1   | 0    | -1   | 1    | -1   | 0    | 0    | 1    | -1   | 0    |

**莫比乌斯变换**

假设 $f(m)$ 和 $g(m)$ 是两个数论函数，具有如下变换
$$
f(m) = \sum_{d \mid m} g(d) \Leftrightarrow g(m) = \sum_{d \mid m} \mu(d) f(\frac{m}{d}).
$$
这种形式下，数论函数 $f(m)$ 称为数论函数 $g(m)$ 的莫比乌斯变换，数论函数 $g(m)$ 称为数论函数 $f(m)$ 的莫比乌斯逆反演。

证明：
$$
\sum_{d \mid m} \mu(d) f(\frac{m}{d}) \overset{f(m) = \sum_{d \mid m} g(d)}{=} 
\sum_{d \mid m} \mu(d) \sum_{k \mid \frac{m}{d}} g(k) \overset{ex. \ order}{=}
\sum_{k \mid m} g(k) \sum_{d \mid \frac{m}{k}} \mu(d) \overset{\sum_{d \mid m} \mu(d) = [m=1]}{=}
g(m).
$$


