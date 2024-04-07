---
title: concrete-math-sums-calculation
date: 2024-03-28 10:25:32
tags: 
---

# 和式的计算 Calculation of Sums

成功处理和式的关键在于，将一个 $\sum$ 改变成另一个更简单或者更接近某个目标的 $\sum$。通过学习一些基本的变换法则并在实践中练习使用它们，就会容易做到这点。

**基本变换法则**

设 $K$ 是任意一个有限整数集合， $K$ 中元素的和式可以使用分配律、结合律、交换律进行变换：
$$
\begin{align*}
\sum_{k \in K} ca_k &= c\sum_{k \in K}a_k; \\
\sum_{k \in K} (a_k + b_k) &= \sum_{k \in K}a_k + \sum_{k \in K}b_k; \\
\sum_{k \in K} a_k &= \sum_{p(k) \in K}a_{p(k)}. \\
\end{align*}
$$
**艾弗森约定**

在和式中可以加入逻辑命题来表示当前项是否加入求和运算，即
$$
\sum_{k \in K} a_k = \sum_{k} a_k[k \in K].
$$
如果 $K$ 和 $K'$ 是整数的任意集合，那么
$$
\sum_{k} a_k[k \in K] + \sum_{k} a_k[k \in K'] = \sum_{k} a_k[k \in K \cap K'] + \sum_{k} a_k[k \in K \cup K'].
$$
**多重和式**

一个和式的项可以用两个或者更多的指标来指定，而不是仅由一个指标来指定，如果 $P(j,k)$ 是 $j$ 与 $k$ 的一种性质，所有使得 $P(j,k)$ 为真的项 $a_{j,k}$ 之和可以用多个求和符号表示，即
$$
\sum_{P(j,k)} a_{j,k} = \sum_{j,k} a_{j,k}[P(j,k)] = \sum_j(\sum_k a_{j,k}[P(j,k)]).
$$
多重和式满足交换求和次序法则，即
$$
\sum_{j \in J}\sum_{k \in K} a_{j,k} = \sum_{j \in J, k \in K} a_{j,k} = \sum_{k \in K}\sum_{j \in J} a_{j,k};\\
\sum_j\sum_k a_{j,k}[P(j,k)] = \sum_{P(j,k)} a_{j,k} = \sum_k\sum_j a_{j,k}[P(j,k)].
$$
多重和式也满足一般交换律，即
$$
\sum_{j \in J, k \in K} a_{j,k} = (\sum_{j \in J} a_j)(\sum_{k \in K} a_k).
$$

## 一般性的方法

下面介绍解决一般情形下求和问题的几种有用策略。

**公式法**

我们可以根据现有的参考资料查找和式的封闭形式解，如
$$
\sum_{0 \leq k \leq n}k^2 = \frac{n(n+1)(2n+1)}{6}.
$$
**数学归纳法**

当我们已经用其他某些不太严格的方法得到了一个封闭形式，然后我们只需要使用归纳法证明它是正确的。上述的式子存在一个等价的公式，即
$$
S_n= \sum_{0 \leq k \leq n}k^2 = \frac{n(n+\frac{1}{2})(n+1)}{3}.
$$
使用归纳法进行证明：

1. 已知 $S_0 = 0 = 0(0+\frac{1}{2})(0+1)/3$，符合初值；
2. 当 $n>0$ 时，假设 $n-1$ 时命题成立；
3. 我们推导得到

$$
\begin{align*}
S_n &= S_{n-1} + n^2 \\
3S_n &= n(n+\frac{1}{2})(n+1) + 3n^2 \\
&= (n^3 - \frac{3}{2}n^2 + \frac{1}{2}n) + 3n^2 \\
&= n^3 + \frac{3}{2}n^2 + \frac{1}{2}n \\
&= n(n+\frac{1}{2})(n+1).
\end{align*}
$$

得证。

**扰动法**

使用扰动法将和式的一项分出去单独计算，可以用来求解封闭形式。假设和式 $S_n = \sum_{0 \leq k \leq n} a_k$，通过将它的最后一项和第一项分离出来，用两种方法重新改写 $S_{n+1}$
$$
\begin{align*}
S_n + a_{n+1} = \sum_{0 \leq k \leq n+1} a_k &= a_0 + \sum_{1 \leq k \leq n+1} a_k \\
&= a_0 + \sum_{1 \leq k+1 \leq n+1} a_{k+1} \\
&= a_0 + \sum_{0 \leq k \leq n+1} a_{k+1} \\
&= a_0 + cS_n.
\end{align*}
$$
将关于 $S_n$ 的方程进行求解，我们就可以得到 $S_n$ 的一般表达式。下面我们使用扰动法求解 $S_n = \sum_{0 \leq k \leq n}k^2$。首先抽取 $S_n$ 的第一项和最后一项，得到关于 $S_n$ 的方程如下
$$
\begin{align*}
S_n + (n+1)^2 &= \sum_{0 \leq k \leq n} (k+1)^2 = \sum_{0 \leq k \leq n} (k^2 + 2k + 1) \\
&= \sum_{0 \leq k \leq n} k^2 + 2 \sum_{0 \leq k \leq n} k + \sum_{0 \leq k \leq n} 1 \\
&= S_n + 2 \sum_{0 \leq k \leq n} k + (n + 1).
\end{align*}
$$
可惜的是，等式两边的 $S_n$ 相互抵消了，但是我们得到了单次求和项 $R_n = \frac{(n+1)^2-(n+1)}{2}$ 的表达式。受此启发，我们期望列出立方求和公式 $T_n = \sum_{0 \leq k \leq n} k^3$ 来获得平方求和公式的解，即
$$
\begin{align*}
T_n + (n+1)^3 &= \sum_{0 \leq k \leq n} (k+1)^3 = \sum_{0 \leq k \leq n} (k^3 + 3k^2 + 3k + 1) \\
&= T_n + 3\sum_{0 \leq k \leq n} k^2 + \frac{3}{2}(n+1)n + (n + 1).
\end{align*}
$$
则我们获得平方求和公式的解 $S_n = \sum_{0 \leq k \leq n} k^2 = \frac{1}{3}n(n+\frac{1}{2})(n+1)$。

**一般公式法**

我们设递归式
$$
\begin{align*}
S_0 &= \alpha; \\
S_n &= S_{n-1} + \beta + \gamma n + \delta n^2, n > 0.
\end{align*}
$$
其解的一般形式为
$$
S_n = A(n)\alpha + B(n)\beta + C(n)\gamma + D(n)\delta
$$
将其套用在平方求和公式中，我们知道 $S_n = S_{n-1} + n^2$，则 $\alpha=\beta=\gamma=0, \delta=1$，即
$$
S_n = A(n)\alpha + B(n)\beta + C(n)\gamma + D(n)\delta = D(n)
$$
首先我们需要知道各个待定系数的表达式

- 令 $S_n = 1$ 就意味着 $\alpha = 1, \beta = \gamma = \delta = 0$，从而 $A(n) = 1$；
- 令 $S_n = n$ 就意味着 $\alpha = \gamma = \delta = 0, \beta = 1$，从而 $B(n) = n$；
- 令 $S_n = n^2$ 就意味着 $\alpha = 0, \beta = -1, \gamma = 2, \delta = 0$，从而 $C(n) = \frac{n^2+n}{2}$；
- 令 $S_n = n^3$就意味着 $\alpha = 0, \beta = 1, \gamma = -3, \delta = 3$，从而 $D(n)=\frac{n(n+\frac{1}{2})(n+1)}{3}$。

接着我们可以得到
$$
S_n = D(n) = \frac{n(n+\frac{1}{2})(n+1)}{3}
$$
**积分法**

利用面积近似的思想，我们可以得出 $\int_0^n x^2 \mathrm{d}x = n^3/3$ 近似于平方求和公式 $S_n = \sum_{0 \leq k \leq n} k^2$。设它们之间的近似误差为 $E_n = S_n - \frac{n^3}{3}$，则 $E_n$ 满足递归式为
$$
\begin{align*}
E_n &= S_n - \frac{n^3}{3} = S_{n-1} + n^2 - \frac{n^3}{3} \\
&= E_{n-1} + \frac{(n-1)^3}{3} + n^2 - \frac{n^3}{3} \\
&= E_{n-1} + n - \frac{1}{3}.
\end{align*}
$$
根据先前的经验，我们可以很快地求出误差项的一般表达式 $E_n = -\frac{n}{3} + \frac{n^2+n}{2}$，从而我们可以得到平方求和公式的一般表达式为
$$
\begin{align*}
S_n &= \int_0^n x^2 \mathrm{d}x + E_n \\
&= \frac{n^3}{3} -\frac{n}{3} + \frac{n^2+n}{2} \\
&= \frac{n(n+\frac{1}{2})(n+1)}{3}.
\end{align*}
$$
**展开和收缩法**

我们可以使用简单的二重和式替换原来的和式，来达到化简的目的。考虑平方求和公式
$$
\begin{align*}
S_n &= \sum_{1 \leq k \leq n}k^2 = \sum_{0 \leq j \leq n} \sum_{0 \leq k \leq n} k \\
&= \sum_{0 \leq j \leq n} \frac{j+n}{2}(n-j+1) \\
&= \frac{1}{2}\sum_{0 \leq j \leq n}(n(n+1)+j-j^2) \\
&= \frac{1}{2}n^2(n+1) + \frac{1}{4}n(n+1) - \frac{1}{2}S_n.
\end{align*}
$$

则我们可以得到
$$
S_n = \frac{2}{3}[\frac{1}{2}n^2(n+1) + \frac{1}{4}n(n+1)] = \frac{n(n+\frac{1}{2})(n+1)}{3}
$$
