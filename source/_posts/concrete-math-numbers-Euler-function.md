---
title: concrete-math-numbers-Euler-function
date: 2024-04-20 14:47:40
tags:
---

# 欧拉函数 Euler's Totient Function

欧拉函数，即 $\varphi(m)$，表示的是小于等于 $m$ 并与 $m$ 互素的数的个数。欧拉将费马小定理推广到非素数的模，称为欧拉定理，如下所示
$$
n^{\varphi(m)} \equiv 1 \pmod m, \ \gcd(n,m)=1 \and n,m \in Z.
$$
类似于费马小定理的证明，我们考虑模 $m$ 意义下的剩余类 $Z_m = \set{r_1,r_2,\dots,r_{\varphi(m)}}$，我们对每个元素乘以 $n$ 构造 $Z_m = \set{nr_1,nr_2,\dots,nr_{\varphi(m)}}$，$Z_m'$ 中每个元素是互不相同的，则两个集合中所有元素的乘积是同余的，即 $r_1 \cdot r_2 \dots r_{\varphi(m)} \equiv n^{\varphi(m)} r_1 \cdot r_2 \dots r_{\varphi(m)} \pmod m$，约去 $r_1 \cdot r_2 \dots r_{\varphi(m)}$ 可得 $n^{\varphi(m)} \equiv 1 \pmod m$。

如果 $m$ 是一个素数幂 $p^k$，则容易计算 $\varphi(m)$。在 $\set{0,1,\dots,p^k-1}$ 中 $p$ 的倍数是 $\set{0,p,2p,\dots,p^k-p}$，共有 $p^{k-1}$ 个，将这些 $p$ 的倍数剔除，余下的元素与 $m$ 互素，共有 $\varphi(p^k) = p^k - p^{k-1}$ 个。特别地，当 $k=1$ 时有 $\varphi(p)=p-1$。

如果 $m$ 不是素数幂，则我们将其进行分解，如 $m= m_1 m_2$，其中 $m_1$ 和 $m_2$ 互素。如果我们能找到 $\varphi(m)$ 与 $\varphi(m_1)$ 和 $\varphi(m_2)$ 的关系，那么我们就可以递归地求出 $\varphi(m)$ 的值。下面我们证明 $\varphi(m) = \varphi(m_1)\varphi(m_2)$，即欧拉函数是积性的。

**证明 1**：假设 $m$ 存在素因数分解 $m=p_1^{a_1} \cdot p_2^{a_2} \cdots p_k^{a_k}$。由于 $\varphi(m)$ 表示的是小于等于 $m$ 并与 $m$ 互素的数的个数，我们假设 $1,2,\dots,m$ 中与 $m$ 有 $t$ 个相同素因子的个数有 $f_t(m)$ 个，那么
$$
\begin{align*}
f_0(m) &= m;\\
f_1(m) &= \sum_{i=1}^k \frac{m}{p_i}; \\
f_2(m) &= \sum_{1 \leq i \leq j \leq k} \frac{m}{p_i p_j}; \\
\cdots \\
f_t(m) &= \sum_{1 \leq i_j \leq k} \frac{m}{\prod_{j=1}^t p_{i_j}}.
\end{align*}
$$
根据容斥原理，我们得到 
$$
\begin{align*}
\varphi(m) &= \sum_{t=0}^k (-1)^t f_t(m) \\
&= m (1 - \sum_{i=1}^k \frac{1}{p_i} + \sum_{1 \leq i \leq j \leq k} \frac{1}{p_i p_j} - \cdots) \\
&= m \prod_{i=1}^k (1 - \frac{1}{p_i}).
\end{align*}
$$
那么，当  $m_1$ 和 $m_2$ 互素时，它们没有大于一的公因子，显然有 $\varphi(m) = \varphi(m_1)\varphi(m_2)$。

**证明 2**：将 $1$ 到 $m_1 m_2$ 排列成如下方阵
$$
\begin{array}{cccccc}
1 & 2 & \cdots & i & \cdots & m_2 \\
1+m_2 & 2+m_2 & \cdots & i+m_2 & \cdots & 2 m_2 \\
1+2 m_2 & 2+2 m_2 & \cdots & i+2 m_2 & \cdots & 3 m_2 \\
\vdots & \vdots & \ddots & \vdots & \ddots & \vdots \\
1+j m_2 & 2+j m_2 & \cdots & i+j m_2 & \cdots & (j+1) m_2 \\
\vdots & \vdots & \ddots & \vdots & \ddots & \vdots \\
1+(m_1-1) m_2 & 2+(m_1-1) m_2 & \cdots & i+(m_1-1) m_2 & \cdots & m_1 m_2
\end{array}
$$
由于 $m_1$ 和 $m_2$ 互素，每一列 $i+jm_2$ 都能构成关于模 $m_2$ 的独立剩余类 $\set{\bar{0},\bar{1},\dots,\overline{m_1 - 1}}$，每一列与 $m_2$ 互素的数有 $\varphi(m_2)$ 个；同理，每一行都能构成关于模 $m_1$ 的独立剩余类 $\set{\bar{0},\bar{1},\dots,\overline{m_2 - 1}}$，每一行与 $m_1$ 互素的数有 $\varphi(m_1)$ 个。对于所有的 $1\leq a < m_1 m_2$，如果有 $\gcd(a,m_1 m_2)=1$，则有 $\gcd(a,m_1) \and \gcd(a,m_2)$，互素的元素个数共有 $\varphi(m) = \varphi(m_1)\varphi(m_2)$ 个。