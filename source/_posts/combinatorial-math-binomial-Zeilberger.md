---
title: combinatorial-math-binomial-Zeilberger
date: 2024-05-05 16:47:14
tags:
---

# Zeilberger 算法 Zeilberger Algorithm

Zeilberger 算法推广了 Gosper 算法，可以机械化地处理所有对 $k$ 求和的和式。其基本思想是，将要求和的项视为两个变量 $n$ 和 $k$ 的一个函数 $t(n,k)$。Zeilberger 算法可以总结为四个步骤

0. 置 $l:=0$。

1. 令 $\hat{t} = \beta_0(n) t(n,k) + \cdots + \beta_l(n) t(n+l,k)$，其中 $\beta_0(n),\dots,\beta_l(n)$ 都是未知的函数。利用 $t(n,k)$ 的性质寻求 $\beta_0(n),\dots,\beta_l(n)$ 的线性组合 $p(n,k)$，其系数是关于 $n$ 和 $k$ 的多项式，使得 $\hat{t}(n,k)$ 可以写成形式 $p(n,k)\bar{t}(n,k)$，其中 $\bar{t}(n,k)$ 就是关于 $k$ 的超几何项。求多项式 $\bar{p}(n,k),q(n,k),r(n,k)$  使得 $\bar{t}(n,k)$ 的项的比值形如 $\frac{\bar{t}(n,k+1)}{\bar{t}(n,k)} = \frac{\bar{p}(n,k+1)}{\bar{p}(n,k)} \frac{q(n,k)}{r(n,k+1)}$，其中 $q(n,k),r(n,k)$ 满足 Gosper 条件。置 $\hat{p}(n,k)=p(n,k)\bar{p}(n,k)$。
2. 置 $\mathrm{deg}(Q)=\mathrm{deg}(q-r), \mathrm{deg}(R)=\mathrm{deg}(q+r)$，以及 $d:=\left\{\begin{array}{l} \mathrm{deg}(\hat{p})-\mathrm{deg}(Q), \quad \mathrm{deg}(Q) \geq \mathrm{deg}(R) \\
   \mathrm{deg}(\hat{p})-\mathrm{deg}(R)+1, \quad \mathrm{deg}(Q) < \mathrm{deg}(R)
   \end{array}\right.$。如果 $d \geq 0$，定义 $s(k)=\alpha_{d}k^{d} + \alpha_{d-1}k^{d-1} + \cdots + \alpha_0, \alpha_d \neq 0$，在方程 $\hat{p}(n,k) = q(n,k)s(n,k+1)-r(n,k)s(n,k)$ 中令 $k$ 的幂次相等，可以得到关于 $\alpha_0,\dots,\alpha_d,\beta_0,\dots,\beta_l$ 的线性方程组。如果这些方程有一个使得 $\beta_0,\dots,\beta_l$ 不全为零的解，则跳转到第 4 步。如若不然，当 $\mathrm{deg}(Q) < \mathrm{deg}(R)$ 且 $\frac{-2\lambda'}{\lambda}$ 是一个大于 $d$ 的整数时（其中 $\lambda$ 是 $q+r$ 中 $k^{\mathrm{deg}(R)-1}$ 的系数，而 $\lambda’$ 是 $q-r$ 中 $k^{\mathrm{deg}(R)-1}$ 的系数），就置 $d:=\frac{-2\lambda'}{\lambda}$ 并重复上述步骤。
3. （项 $\hat{t}(n,k)$ 不是超几何可求和的。）将 $l$ 增加 1 并退回第 1 步。
4. （成功。）置 $T(n,k):=\frac{r(n,k)s(n,k)\bar{t}(n,k)}{\bar{p}(n,k)}$。由该算法，就得到了 $\hat{t}(n,k) = T(n,k+1) - T(n,k)$。

**例 1**：计算 $\sum \begin{pmatrix} n \\ k \end{pmatrix} z^k$。

令 $t(n,k) = \begin{pmatrix} n \\ k \end{pmatrix} z^k$，由于
$$
\begin{align*}
\frac{t(n+1, k)}{t(n, k)} & =\frac{(n+1)!z^{k}}{(n+1-k)!k!} \frac{(n-k)!k!}{n!z^{k}} \\
& =\frac{n+1}{n+1-k}.
\end{align*}
$$
我们有
$$
\hat{t}(n,k) = p(n,k)\frac{t(n,k)}{n+1-k}.
$$
其中
$$
p(n,k) = (n+1-k)\beta_0(n) + (n+1)\beta_1(n).
$$
令 $\bar{t}(n,k)=\hat{t}(n,k)/p(n,k), \bar{p}(n,k)=\hat{p}(n,k)/p(n,k)$，应用 Gosper 算法
$$
\frac{\bar{t}(n,k+1)}{\bar{t}(n,k)} = \frac{(n+1-k)z}{k+1} = \frac{\bar{p}(n,k+1)}{\bar{p}(n,k)} \frac{q(n,k)}{r(n,k+1)}.
$$
如果 $\bar{p}(n,k)=1$，则有 $q(n,k) = (n+1-k)z, r(n,k) = k$。由于 $\mathrm{deg}(Q) = \mathrm{deg}(R) = 1$，则有 $d = \mathrm{deg}(\hat{p}) - \mathrm{deg}(Q) = 0$，设 $s(n,k)=\alpha_0(n)$，则方程 $\hat{p}(n,k)=q(n,k)s(n,k+1)-r(n,k)s(n,k)$ 变为
$$
(n+1-k)\beta_0(n) + (n+1)\beta_1(n) = (n+1-k)z\alpha_0(n) - k\alpha_0(n).
$$
令 $k$ 的幂次相等，就得到不含 $k$ 的方程组
$$
\begin{align*}
(n+1)\beta_0(n)+(n+1)\beta_1(n)-(n+1-k)z\alpha_0(n) &= 0, \\
-\beta_0(n) + (z+1)\alpha_0(n) &= 0.
\end{align*}
$$
因此有一个满足条件 $\beta_0(n)=z+1, \beta_1(n)=-1, \alpha_0(n) = 1$ 的解。则
$$
T(n,k) = \frac{r(n,k)s(n,k)\bar{t}(n,k)}{\bar{p}(n,k)} = \frac{k}{n+1-k}t(n,k) = \begin {pmatrix} n \\ k-1 \end{pmatrix} z^k.
$$
**例 2**：找出 $S(n) = \sum_k \begin{pmatrix} n \\ 2k \end{pmatrix}$ 的递归关系。

我们有
$$
\frac{t(n+1,k)}{t(n,k)} = \frac{n+1}{n-2k+1}.
$$
因此
$$
\hat{t}(n,k)=p(n,k) \frac{t(n,k)}{n-2k+1}.
$$
其中
$$
p(n,k) = (n - 2k + 1)\beta_0(n) + (n+1)\beta_1(n).
$$
令 $\bar{t}(n,k) = \frac{\hat{t}(n,k)}{p(n,k)}, \bar{p}(n,k) = \frac{\hat{p}(n,k)}{p(n,k)}$，考虑
$$
\frac{\bar{t}(n,k+1)}{\bar{t}(n,k)} = \frac{\bar{p}(n,k+1)}{\bar{p}(n,k)} \frac{q(n,k)}{r(n,k+1)} = \frac{(n-2k)(n-2k+1)}{(2k+1)(2k+2)}
$$
如果 $\bar{p}(n,k) = 1$，那么 $q(n,k) = (n-2k)(n-2k+1), r(n,k) = 2k(2k-1) \Rightarrow d = \mathrm{deg}(\hat{p}) - \mathrm{deg}(R) + 1 = 0$，设 $s(n,k) = s$，有 $\beta_0(n) = 2ns, \beta_1(n) = -ns$，因此
$$
2ns \hat{t(n,k)} - ns \hat{t(n+1,k)} = 0 \Rightarrow 2S_n = S_{n+1}.
$$
