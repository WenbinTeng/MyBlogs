---
title: concrete-math-binomial-Gosper
date: 2024-05-05 16:46:52
tags:
---

# Gosper 算法 Gosper Algorithm

Gosper 算法适用于求和项是超几何项的求和，其一般分为两个步骤。

第一步是将项的比值表示成一个特殊的形式
$$
\frac{t(k+1)}{t(k)} = \frac{p(k+1)}{p(k)}\frac{q(k)}{r(k+1)}.
$$
其中 $p,q,r$ 是满足以下条件的多项式
$$
(k+\alpha) | q(k) \and (k+\beta) | r(k) \Rightarrow \alpha - \beta \notin Z.
$$
可以暂时从 $p(k)=1$ 出发，并令 $q(k)$ 和 $r(k)$ 是将它们分解成线性因子后，项的比值的分子和分母。如果 $q$ 和 $r$ 有因子 $(k+\alpha)$ 和 $(k+\beta)$，其中 $\alpha-\beta=N>0$，则将其从 $q$ 和 $r$ 中去除，并用 $p(k)(k+\alpha-1)^{\underline{N-1}}$ 代替 $p(k)$。重复这个过程直至上式成立。

第二步是求一个超几何项 $T(k)$ 使得 $t(k)=T(k+1)-T(k)$。这一过程并不明显。首先将 $T(k)$ 写成未知形式
$$
T(k) = \frac{r(k)s(k)t(k)}{p(k)}.
$$
其中 $s(k)$ 就是一个未知的函数，其满足关系
$$
p(k) = q(k)s(k+1)-r(k)s(k).
$$
当 $p(k),q(k),r(k)$ 是给定的多项式时，则我们就需要求出多项式 $s(k)$ 或证明其不存在。当 $s(k)$ 有特定的次数 $d$ 时，其可以表示为
$$
s(k)=\alpha_{d}k^{d} + \alpha_{d-1}k^{d-1} + \cdots + \alpha_0, \alpha_d \neq 0.
$$
那么我们如何确定 $s(k)$ 的次数 $d$ 呢？其至多存在两种可能性。我们令 $Q(k)=q(k)-r(k), R(k)=q(k)+r(k)$，$\lambda$ 为 $R(k)$ 的最高项系数，$\lambda'$ 为 $Q(k)$ 的 $\mathrm{deg}(R) - 1$ 项系数，则

- 如果 $\mathrm{deg}(Q) \geq \mathrm{deg}(R)$，那么 $d=\mathrm{deg}(p) - \mathrm{deg}(Q)$；
- 如果 $\mathrm{deg}(Q) < \mathrm{deg}(R)$，那么 $d = \mathrm{deg}(p) - \mathrm{deg}(R) + 1$ 或 $d=\frac{-2\lambda'}{\lambda}$，仅当后者是大于前者的整数时才考虑后者。

至此我们求出了多项式 $s(k)$，进而求得了 $T(k)$ 的表达式，因此
$$
\sum_{k=0}^n t_k = \sum_{k=0}^n (T(k+1) - T(k)) = T(n+1) - T(0).
$$
**例 1**：计算 $\sum \begin{pmatrix} n \\ k \end{pmatrix} (-1)^k \delta k$。

令 $t(k) = \begin{pmatrix} n \\ k \end{pmatrix} (-1)^k = \frac{n!(-1)^k}{k!(n-k)!}$，我们将相邻项的比值表示为要求的形式，即
$$
\frac{t(k+1)}{t(k)} = \frac{k-n}{k+1} = \frac{p(k+1)}{p(k)}\frac{q(k)}{r(k+1)}.
$$
这里我们直接取 $p(k)=1,q(k)=k-n,r(k)=1$，这种选取要求满足条件。

现在考虑 $Q(k)=-n,R(k)=2k-n$，由于 $\mathrm{deg}(Q) < \mathrm{deg}(R)$，要么 $d = \mathrm{deg}(p) - \mathrm{deg}(R) + 1 = 0$，要么 $d=\frac{-2\lambda'}{\lambda} = n$，我们先考虑前者。假设 $d = 0$，则 $s(k)=\alpha_0$，代入 $p(k)$ 的表达式有
$$
1=(k-n)\alpha_0 - k\alpha_0.
$$
解得 $\alpha_0 = -\frac{1}{n}$，则 $T(k) = \frac{r(k)s(k)t(k)}{p(k)} = k(-\frac{1}{n})\begin{pmatrix} n \\ k \end{pmatrix} (-1)^k = \begin{pmatrix} n-1 \\ k-1 \end{pmatrix} (-1)^{k-1}$ 。

如果我们计算求和 $\sum_k \begin{pmatrix} n \\ k \end{pmatrix} (-1)^k \delta k = T(n+1)-T(1)=0$，与使用二项式定理得出的结论一致。

**例 2**：计算 $\sum \frac{\delta k}{k^3 - k}$。

令 $t_k = \frac{1}{k^3-k}$，则
$$
\frac{t_{k+1}}{t_k} = \frac{k^3-k}{(k+1)^3 - (k+1)} = \frac{k-1}{k+2} = \frac{p(k+1)q(k)}{p(k)r(k+1)}.
$$
如果 $p(k) = 1, q(k) = k - 1, r(k) = k + 1$，其满足条件。考虑
$$
\begin{align*}
    Q(k) &= q(k) - r(k) = -2, \\
    R(k) &= q(k) + r(k) = 2k.
\end{align*}
$$
有 $d = \mathrm{deg}(p) - \mathrm{deg}(R) + 1 = 0 \Rightarrow s(k) = \alpha_0$。由 $p(k) = q(k)s(k+1) - r(k)s(k)$ 解得 $\alpha_0 = -\frac{1}{2}$。则
$$
\begin{align*}
    T(k) = \frac{r(k)s(k)t(k)}{p(k)} = -\frac{1}{2k(k-1)}.
\end{align*}
$$
因此
$$
\sum \frac{\delta k}{k^3 - k} = -\frac{1}{2k(k-1)} + C.
$$
