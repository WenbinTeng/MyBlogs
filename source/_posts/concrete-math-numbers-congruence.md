---
title: concrete-math-numbers-congruence
date: 2024-04-18 21:11:11
tags:
---

# 同余数 Congruence

当两个整数 $a$ 与 $b$ 关于整数 $m$ 的取余运算结果相同，我们称为 $a$ 关于模 $m$ 与 $b$ 同余，记作
$$
a \equiv b \pmod m \Leftrightarrow a \bmod m = b \bmod m.
$$
**定理 1**：$a \equiv b \pmod m \Leftrightarrow m|(a-b)$

证明：假设 $a = k_1 m + r_1$ 与 $b = k_2 m + r_2$， 那么 $a-b = (k_1 - k_2)m + (r_1 - r_2)$，当 $a \equiv b \pmod m$ 时，有 $r_1 = r_2$，所以 $a-b = (k_1 - k_2)m$ 必定可以被 $m$ 整除。反向证明同理。

**定理 2**：$a \equiv b \pmod m \Leftrightarrow a = b + km$

证明：假设 $a = k_1 m + r_1$ 与 $b = k_2 m + r_2$，已知 $a \equiv b \pmod m$，则 $r_1 = r_2$，有  $a = k_1 m + r_1 = (k_2+k) m + r_2 = (k_2 m + r_2) + km = b + km$。反向证明同理。

同余数具有以下性质

- 自反性：$a \equiv a \pmod m $；
- 对称性：$a \equiv b \pmod m \Rightarrow b \equiv a \pmod m$；
- 传递性：$a \equiv b \pmod m \and b \equiv c \pmod m \Rightarrow a \equiv c \pmod m$；
- 同加性：$a \equiv b \pmod m \Rightarrow a+c \equiv b+c \pmod m$，$a \equiv b \pmod m \and c \equiv d \pmod m \Rightarrow a+c \equiv b+d \pmod m$；
- 同乘性：$a \equiv b \pmod m \Rightarrow ac \equiv bc \pmod m$，$a \equiv b \pmod m \and c \equiv d \pmod m \Rightarrow ac \equiv bd \pmod m$；
- 同幂性：$a \equiv b \pmod m \Rightarrow a^n \equiv b^n \pmod m$
- $a \bmod p = a \bmod q = x \and \gcd(p,q) = 1 \Rightarrow a \pmod{pq}$。

**中国剩余定理**

对于一元线性同余方程组
$$
\left\{ \begin{matrix}
x \equiv a_1 \pmod {p_1} \\
x \equiv a_2 \pmod {p_2} \\
\cdots \\
x \equiv a_n \pmod {p_n}
\end{matrix} \right.
$$
一般性的求解过程是

- 求所有模数的积 $p = p_1 \cdot p_2 \cdots p_n$；
- 对于第 $i$ 个方程：
  - 计算 $m_i = \frac{p}{p_i}$；
  - 计算 $m_i$ 在模 $p_i$ 下的逆元 $m_i^{-1}$；
  - 计算 $c_i = m_i m_i^{-1}$。
- 方程组在模 $p$ 意义下的唯一解为 $x = \sum_{i=1}^k a_i c_i$。

例题：有物不知其数，三三数之剩二，五五数之剩三，七七数之剩二。问物几何？
$$
\left\{ \begin{matrix}
x \equiv 2 \pmod {3} \\
x \equiv 3 \pmod {5} \\
x \equiv 2 \pmod {7}
\end{matrix} \right.
$$
首先，我们计算模数的积为 $p = 3 \times 5 \times 7 = 105$。然后，我们求解方程
$$
\left\{ \begin{matrix}
x \equiv 1 \pmod {3} \\
x \equiv 0 \pmod {5} \\
x \equiv 0 \pmod {7}
\end{matrix} \right.
$$
由上述方程我们知道， $5 \mid x \and 7 \mid x \Rightarrow 35 \mid x$，显然，$x_1 = 70$ 是最小正整数解。同理，我们求得 $x_2 = 21, x_3 = 15$ 分别为方程 $\left\{ \begin{matrix}
x \equiv 0 \pmod {3} \\
x \equiv 1 \pmod {5} \\
x \equiv 0 \pmod {7}
\end{matrix} \right.$ 与 $\left\{ \begin{matrix}
x \equiv 0 \pmod {3} \\
x \equiv 0 \pmod {5} \\
x \equiv 1 \pmod {7}
\end{matrix} \right.$ 的最小正整数解。最后，我们得到 $x = 70a_1 + 21a_2 + 15a_3 \pmod{105}$ 为方程的通解，将 $a_1 = 2, a_2 = 3, a_3 = 2$ 代入后得到方程的解 $x = 70 \times 2 + 21 \times 3 + 15 \times 2 \pmod{105} = 23$。

