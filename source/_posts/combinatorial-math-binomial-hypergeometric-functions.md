---
title: combinatorial-math-binomial-hypergeometric-functions
date: 2024-04-17 08:55:12
tags:
---

# 超几何函数 Hypergeometric Functions

超几何技术是研究二项式系数之和的系统方法的基础，一般的超几何级数是关于 $z$ 且带有 $m+n$ 个参数的幂级数，它用上升的阶乘幂定义如下
$$
F\left(\left.\begin{array}{c}
a_{1}, \cdots, a_{m} \\
b_{1}, \cdots, b_{n}
\end{array} \right\rvert\, z\right)=\sum_{k \geq 0} \frac{a_{1}^{\bar{k}} \cdots a_{m}^{\bar{k}} z^{k}}{b_{1}^{\bar{k}} \cdots b_{n}^{\bar{k}} k!}.
$$
许多重要的函数都作为一般的超几何级数的特例出现，如
$$
F\left(\left.\begin{array}{c}
1 \\
1
\end{array} \right\rvert\, z\right)=\sum_{k \geq 0} \frac{z^k}{k!} = e^z.
$$

$$
F\left(\left.\begin{array}{c}
1,1 \\
1
\end{array} \right\rvert\, z\right)=\sum_{k \geq 0} z^k = \frac{1}{1-z}.
$$

**高斯超几何函数**
$$
F\left(\left.\begin{array}{c}
a, b \\
c
\end{array} \right\rvert\, z\right)=\sum_{k \geq 0} \frac{a^{\bar{k}} b^{\bar{k}} z^{k}}{c^{\bar{k}} k!}.
$$
上述超几何级数被称为高斯超几何函数，大部分函数都可以使用这个函数 $F(a,b;c;x)$ 进行表示。最常用的三个高斯超几何函数为
$$
F\left(\left.\begin{array}{c}
a, b \\
c
\end{array} \right\rvert\, 1\right)= \frac{\Gamma(c-a-b)\Gamma(c)}{\Gamma(c-a)\Gamma(c-b)}.
$$

$$
F\left(\left.\begin{array}{c}
a, -n \\
c
\end{array} \right\rvert\, 1\right)= \frac{(c-a)^{\bar{n}}}{c^{\bar{n}}} = \frac{(a-c)^{\underline{n}}}{(-c)^{\underline{n}}}.
$$

$$
F\left(\left.\begin{array}{c}
a, b \\
1+b-a
\end{array} \right\rvert\, -1\right)= \frac{(b/2)!}{b!}(b-a)^{\underline{b/2}}.
$$

我们的目标是，将求和公式使用上述三个高斯超几何函数之一进行表示，从而达到化简的目的。实际上高斯超几何函数不止以上几种，但是其他形式都比较复杂。如果和式转换到了其他的高斯超几何函数的形式，计算难度未必会降低。那么什么样的和式能转换为高斯超几何函数呢？

- 超几何函数在 $z=0$ 时值为 $1$，即 $F\left(\left.\begin{array}{c}a, b \\c\end{array} \right\rvert\, 0\right)=1$。
- 相邻两项的值为有理函数，即 $\frac{t_{k+1}}{t_{k}} =\frac{a_{1}^{\overline{k+1}} \cdots a_{m}^{\overline{k+1}}}{a_{1}^{\bar{k}} \cdots a_{m}^{\bar{k}}} \frac{b_{1}^{\bar{k}} \cdots b_{n}^{\bar{k}}}{b_{1}^{\overline{k+1}} \cdots b_{n}^{\overline{k+1}}} \frac{k!}{(k+1)!} \frac{z^{k+1}}{z^{k}}
  =\frac{\left(k+a_{1}\right) \cdots\left(k+a_{m}\right) z}{\left(k+b_{1}\right) \cdots\left(k+b_{n}\right)(k+1)}$
- 相邻两项的值是关于 $k$ 的函数而不是固定值。

使用高斯超几何函数化简和式的一般步骤为：

- 计算和式相邻两项的比值 $\frac{t_{k+1}}{t_k}$；
- 将比值表示为 $\frac{t_{k+1}}{t_k} = \frac{(k+a)(k+b)}{(k+c)(k+1)}$ 的形式，将其参数与高斯超几何函数形式 $F\left(\left.\begin{array}{c}a, b \\c\end{array} \right\rvert\, z\right)$ 对应；
- 计算和式首项 $t_0$；
- 代入高斯超几何函数形式化简和式 $\sum_k t_k = t_0 \cdot F\left(\left.\begin{array}{c}a, b \\c\end{array} \right\rvert\, z\right)$。

**例 1**：计算 $\sum_{k\leq n} \begin{pmatrix} r+k \\ k \end{pmatrix}$。

首先将变换原式的求和范围 $\sum_{k\leq n} \begin{pmatrix} r+k \\ k \end{pmatrix} = \sum_k \begin{pmatrix} r+n-k \\ n-k \end{pmatrix}$，令 $t_k = \begin{pmatrix} r+n-k \\ n-k \end{pmatrix}$，则相邻两项的比值为 $\frac{t_{k+1}}{t_k} = \begin{pmatrix} r+n-k-1 \\ n-k-1 \end{pmatrix} / \begin{pmatrix} r+n-k \\ n-k \end{pmatrix} = \frac{n-k}{r+n-k}$。

然后将其转换为高斯超几何函数的形式，$\frac{t_{k+1}}{t_k} = \frac{(k-n)(k+1)}{(k-r-n)(k+1)}$，得到对应的形式为$F\left(\left.\begin{array}{c} -n, 1 \\ -n-r \end{array} \right\rvert\, 1\right)$。

接着计算首项 $t_0 = \begin{pmatrix} r+n \\ n \end{pmatrix}$。

最后代入高斯超几何函数公式 
$$
\begin{align*}
\sum_{k\leq n} \begin{pmatrix} r+k \\ k \end{pmatrix}
&= \begin{pmatrix} r+n \\ n \end{pmatrix} F\left(\left.\begin{array}{c} -n, 1 \\ -n-r \end{array} \right\rvert\, 1\right) \\
&= \begin{pmatrix} r+n \\ n \end{pmatrix} \frac{(1+n+r)^{\underline{n}}}{(n+r)^{\underline{n}}} \\
&= \frac{(r+n)!}{r!n!}\frac{1+n+r}{r+1} \\
&= \frac{(n+r+1)!}{n!(r+1)!} \\
&= \begin{pmatrix} n+r+1 \\ n \end{pmatrix}.
\end{align*}
$$
**例 2**：计算 $\sum_k \begin{pmatrix} r \\ k \end{pmatrix} \begin{pmatrix} s \\ n-k \end{pmatrix}$。

计算相邻两项的比值 $\frac{t_{k+1}}{t_k} = \frac{(k-r)(k-n)}{(k+1)(k+s-n+1)}$，对应的高斯超几何函数的形式为 $F\left(\left.\begin{array}{c} -r, -n \\ s-n+1 \end{array} \right\rvert\, 1\right)$，首项 $t_0 = \begin{pmatrix} s \\ n \end{pmatrix}$，则 $\sum_k \begin{pmatrix} r \\ k \end{pmatrix} \begin{pmatrix} s \\ n-k \end{pmatrix} = \begin{pmatrix} s \\ n \end{pmatrix} F\left(\left.\begin{array}{c} -r, -n \\ s-n+1 \end{array} \right\rvert\, 1\right) = \begin{pmatrix} r+s \\ n \end{pmatrix}$。

**例 3**：计算 $\sum_k \begin{pmatrix} m \\ k+n \end{pmatrix} \begin{pmatrix} k+n \\ 2k \end{pmatrix} 4^k$。

我们令
$$
t_k = \begin{pmatrix} m \\ k+n \end{pmatrix} \begin{pmatrix} k+n \\ 2k \end{pmatrix} 4^k =
\frac{m!}{(m-k-n)!(n-k)!(2k)!} 4^k
$$

$$
\Rightarrow \frac{t_{k+1}}{t_k} = \frac{(m-n-k)(n-k)}{(k+\frac{1}{2})(k+1)}.
$$

则
$$
\begin{align*}
\sum_k \begin{pmatrix} m \\ k+n \end{pmatrix} \begin{pmatrix} k+n \\ 2k \end{pmatrix} 4^k &= \begin{pmatrix} m \\ n \end{pmatrix} F\left(\left.\begin{array}{c}n-m, n \\\frac{1}{2}\end{array} \right\rvert\, 1\right) \\
&= \begin{pmatrix} m \\ n \end{pmatrix} \frac{(\frac{1}{2}-n+m)^{\bar{n}}}{(\frac{1}{2})^{\bar{n}}} \\
&= \begin{pmatrix} m \\ n \end{pmatrix} \frac{\begin{pmatrix} m-\frac{1}{2} \\ n \end{pmatrix}}{\begin{pmatrix} n-\frac{1}{2} \\ n \end{pmatrix}} \\
&= \begin{pmatrix} 2m \\ 2n \end{pmatrix}.
\end{align*}
$$
