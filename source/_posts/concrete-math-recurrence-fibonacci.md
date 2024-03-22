---
title: concrete-math-recurrence-fibonacci
date: 2024-03-19 19:39:32
tags:
---

# 斐波那契数 Fibonacci Numbers

**问题定义：**对于一个数列：$1,1,2,3,5,8,13,21,34,55,89\dots$，从第三项开始，每一项都等于前两项之和，设第 $n$ 项为 $F_n$。

显然，该问题的递归关系为
$$
\begin{align*}
F_0 &= 1, \\
F_1 &= 1, \\
F_n &= F_{n-1} + F_{n-2}, n>1.
\end{align*}
$$

## 封闭形式求解

我们可以使用三种方法求解斐波那契数递归关系的封闭形式：等比数列求解法、特征方程法和生成函数法。

### 等比数列求解

首先我们使用待定系数法构造符合原递归关系的等比数列，设
$$
F_n - \lambda F_{n-1} = \mu (F_{n-1} - \lambda F_{n-2}).
$$
对比原递归关系的系数，有
$$
\left\{
\begin{aligned}
\lambda + \mu &= 1, \\
-\lambda \times \mu &= 1.
\end{aligned}
\right.
$$
解得
$$
\left\{
\begin{aligned}
\lambda = \frac{1+\sqrt{5}}{2}, \\
\mu = \frac{1-\sqrt{5}}{2},
\end{aligned}
\right.
\ or \
\left\{
\begin{aligned}
\lambda = \frac{1-\sqrt{5}}{2}, \\
\mu = \frac{1+\sqrt{5}}{2}.
\end{aligned}
\right.
$$
代入原式，得到等比数列通项公式
$$
\left\{
\begin{aligned}
F_n - \frac{1+\sqrt{5}}{2} F_{n-1} &= \frac{1-\sqrt{5}}{2} (F_{n-1} - \frac{1+\sqrt{5}}{2} F_{n-2}) = (\frac{1-\sqrt{5}}{2})^{n-2}(F_1 - \frac{1-\sqrt{5}}{2}F_0), \\
F_n - \frac{1-\sqrt{5}}{2} F_{n-1} &= \frac{1+\sqrt{5}}{2} (F_{n-1} - \frac{1-\sqrt{5}}{2} F_{n-2}) = (\frac{1+\sqrt{5}}{2})^{n-2}(F_1 - \frac{1+\sqrt{5}}{2}F_0).
\end{aligned}
\right.
$$
两式分别乘以 $\frac{1-\sqrt{5}}{2}$ 和 $\frac{1+\sqrt{5}}{2}$ 后相减，得到
$$
F_n = \frac{1}{\sqrt{5}}[(\frac{1+\sqrt{5}}{2})^n-(\frac{1-\sqrt{5}}{2})^n].
$$

### 特征方程求解

问题的递归关系是齐次的，其特征方程为
$$
x^2 = x + 1.
$$
得到两个特征根 $x_1=\frac{1+\sqrt{5}}{2}$ 和 $x_2=\frac{1+\sqrt{5}}{2}$，则特征方程的通解为
$$
x = C_1x_1^n + C_2x_2^n.
$$
将递推关系的初值代入，得到 $C_1=\frac{1}{\sqrt{5}}$ 与 $C_2=-\frac{1}{\sqrt{5}}$，则
$$
x = \frac{1}{\sqrt{5}}[(\frac{1+\sqrt{5}}{2})^n-(\frac{1-\sqrt{5}}{2})^n].
$$

### 生成函数求解

设生成函数
$$
g(x) = F_0 + F_1x + F_2x^2 + \cdots + F_nx^n + \cdots
$$
利用给定的递归关系和初值 $F_0=1$，我们得到
$$
\begin{align*}
g(x) - (x+x^2)g(x) &= F_0 + (F_1-F_0)x + (F_2-F_1-F_0)x^2 + (F_3-F_2-F_1)x^3 + \cdots + (F_n-F_{n-1}-F_{n-2})x^n + \cdots \\
&= 1.
\end{align*}
$$
因此
$$
\begin{align*}
g(x) &= \frac{1}{1-x-x^2} \\
&= \frac{1}{\sqrt{5}}(\frac{1}{1-\frac{1+\sqrt{5}}{2}x}-\frac{1}{1-\frac{1-\sqrt{5}}{2}x}) \\
&= \frac{1}{\sqrt{5}}(\sum_{n=0}^{\infty}(\frac{1+\sqrt{5}}{2})^nx^n-\sum_{n=0}^{\infty}(\frac{1-\sqrt{5}}{2})^nx^n) \\
&= \sum_{n=0}^{\infty}\frac{1}{\sqrt{5}}((\frac{1+\sqrt{5}}{2})^n-(\frac{1-\sqrt{5}}{2})^n)x^n.
\end{align*}
$$
我们得到递推关系的解为
$$
F_n = \frac{1}{\sqrt{5}}((\frac{1+\sqrt{5}}{2})^n-(\frac{1-\sqrt{5}}{2})^n).
$$
