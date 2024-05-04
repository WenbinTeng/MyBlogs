---
title: combinatorial-math-binomial-identities
date: 2024-04-10 09:24:35
tags:
---

# 基本二项恒等式 Basic Binomial Identities

在组合意义中，$\begin{pmatrix} n \\ k \end{pmatrix}$ 代表从具有 $r$ 个元素的集合中选取 $k$ 个元素作为子集的方法数，其计算公式为
$$
\begin{pmatrix} r \\ k \end{pmatrix} = \begin{align*} \left\{ \begin{matrix}
\frac{r(r-1)\cdots (r-k+1)}{k(k-1)\cdots (1)} = \frac{r!}{k!(r-k)!} = \frac{r^{\underline{k}}}{k!}, &k \geq 0, k \in Z, \\
0, &k < 0, k \in Z.
\end{matrix} \right. \end{align*}
$$
**对称恒等式**
$$
\begin{pmatrix} r \\ k \end{pmatrix} = 
\frac{n!}{k!(n-k)!} =
\frac{n!}{(n-(n-k))!(n-k)!} =
\begin{pmatrix} r \\ n-k \end{pmatrix}, n,k \in Z, n \geq 0.
$$
**吸收恒等式**
$$
\begin{pmatrix} r \\ k \end{pmatrix} = \frac{r}{k} \begin{pmatrix} r-1 \\ k-1 \end{pmatrix}, k \in Z.
$$
**相伴恒等式**
$$
(r-k)\begin{pmatrix} r \\ k \end{pmatrix} = r \begin{pmatrix} r-1 \\ k \end{pmatrix}.
$$
<u>**加法公式**</u>
$$
\begin{pmatrix} r \\ k \end{pmatrix} =
\begin{pmatrix} r-1 \\ k \end{pmatrix} +
\begin{pmatrix} r-1 \\ k-1 \end{pmatrix},
k \in Z.
$$
**上指标求和**
$$
\sum_{0 \leq k \leq n} \begin{pmatrix} k \\ m \end{pmatrix} = 
\begin{pmatrix} 0 \\ m \end{pmatrix} +
\begin{pmatrix} 1 \\ m \end{pmatrix} + \cdots +
\begin{pmatrix} n \\ m \end{pmatrix} =
\begin{pmatrix} n+1 \\ m+1 \end{pmatrix}, m,n \in Z, m,n \geq 0.
$$
**上指标反转**
$$
\begin{pmatrix} r \\ k \end{pmatrix} = (-1)^k \begin{pmatrix} k-r-1 \\ k \end{pmatrix}, k \in Z.
$$
**平行求和**
$$
\sum_{k \leq n} \begin{pmatrix} r+k \\ k \end{pmatrix} = \begin{pmatrix} r+n+1 \\ k \end{pmatrix}, n \in Z.
$$
**二项式定理**
$$
(x+y)^r = \sum_k \begin{pmatrix} r \\ k \end{pmatrix} x^k y^{r-k}.
$$
**多项式系数**
$$
\begin{align*}
\begin{pmatrix} a_1+a_2+\cdots+a_m \\ a_1,a_2,\cdots,a_m \end{pmatrix} &=
\frac{a_1+a_2+\cdots+a_m}{a_1!a_2! \cdots a_m!} \\
&= \begin{pmatrix} a_1+a_2+\cdots+a_m \\ a_2+\cdots+a_m \end{pmatrix} \cdots \begin{pmatrix} a_{m-1}+a_m \\ a_m \end{pmatrix}.

\end{align*}
$$
**范德蒙德卷积**
$$
\sum_k \begin{pmatrix} r \\ k \end{pmatrix} \begin{pmatrix} s \\ n-k \end{pmatrix} = \begin{pmatrix} r+s \\ n \end{pmatrix}, n \in Z.
$$
