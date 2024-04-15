---
title: concrete-math-integers-sums
date: 2024-04-14 21:42:33
tags:
---

# 底和顶的和式 Floor and Ceiling Sums

通常在含有取整的和式中，求封闭形式解的行之有效的技巧是通过引入一个新的变量来规避底或顶。

**例 1**：求 $\sum_{0 \leq k \leq n} \lfloor \sqrt{k} \rfloor$。

令 $m=\lfloor \sqrt{k} \rfloor$，则
$$
\begin{align*}
\sum_{0 \leq k \leq n} \lfloor \sqrt{k} \rfloor &= \sum_{k,m \geq 0} m[k<n][m=\lfloor \sqrt{k} \rfloor] \\
&= \sum_{k,m \geq 0} m[k<n][m \leq \sqrt{k} < m+1] \\
&= \sum_{k,m \geq 0} m[k<n][m^2 \leq k < (m+1)^2] \\
&= \sum_{k,m \geq 0} m[m^2 \leq k < (m+1)^2 \leq n] +
\sum_{k,m \geq 0} m[m^2 \leq k < n < (m+1)^2].
\end{align*}
$$
我们假设 $n=a^2$，则加号右侧的式子条件为空，求和结果为零。加号左侧的式子为
$$
\begin{align*}
\sum_{k,m \geq 0} m[m^2 \leq k < (m+1)^2 \leq n] &= \sum_{m \geq 0} m((m+1)^2-m^2)[m+1 \leq a] \\
&= \sum_{m \geq 0} m(2m+1)[m<a] \\
&= \sum_{m \geq 0} (2m^{\underline{2}}+3m^{\underline{1}})[m<a] \\
&= \sum_0^a (2m^{\underline{2}}+3m^{\underline{1}}) \delta m \\
&= \frac{2}{3} a(a-1)(a-2)+\frac{3}{2} a(a-1) \\
&= \frac{1}{6}(4 a+1) a(a-1).
\end{align*}
$$
一般情形下，我们假设 $a = \lfloor \sqrt{n} \rfloor$，加号左侧的式子求解过程同上。加号右侧的式子为
$$
\sum_{k,m \geq 0} m[m^2 \leq k < n < (m+1)^2] = (n-a^2)a.
$$
则所求封闭形式的解为
$$
\sum_{0 \leq k \leq n} \lfloor \sqrt{k} \rfloor = na-\frac{1}{3}a^3-\frac{1}{2}a^2-\frac{1}{6}a,a = \lfloor \sqrt{n} \rfloor.
$$
**例 2**：求 $\sum_{0 \leq k < m} \lfloor \frac{nk+x}{m} \rfloor$。

通过枚举 $m=1,2,\dots$，可以发现和式满足形式
$$
a \lfloor \frac{x}{a} \rfloor + bn + c.
$$
我们对式子做以下变换
$$
\lfloor \frac{nk+x}{m} \rfloor = \lfloor \frac{(nk+x) \bmod m}{m} \rfloor + \frac{kn}{m} - \frac{kn \bmod m}{m}
$$
对第一项，通过枚举后发现其值是周期性的，其周期重复次数是 $d=\gcd(m,n)$，则第一项求和的值为
$$
\begin{align*}
\sum_{0 \leq k < m} \lfloor \frac{(nk+x) \bmod m}{m} \rfloor &= \lfloor \frac{x \bmod m}{m} \rfloor + \lfloor \frac{(n+x) \bmod m}{m} \rfloor + \cdots + \lfloor \frac{(n(m-1)+x) \bmod m}{m} \rfloor \\
&= d(\overbrace{\lfloor \frac{x}{m} \rfloor + \lfloor \frac{(x+d)}{m} \rfloor + \cdots + \lfloor \frac{(x+m-d)}{m} \rfloor}^m) \\
&= d(\overbrace{\lfloor \frac{x/d}{m/d} \rfloor + \lfloor \frac{x/d+1}{m/d} \rfloor + \cdots + \lfloor \frac{x/d+m/d-1}{m/d} \rfloor}^m) \\
&= d \lfloor \frac{x}{d} \rfloor.
\end{align*}
$$
对第二项求和有
$$
\sum_{0 \leq k < m}\frac{kn}{m} = \frac{(m-1)n}{2}.
$$
对第三项，其值同样有周期性，其求和的值为
$$
\sum_{0 \leq k < m} \frac{kn \bmod m}{m} = d(\frac{0}{m} + \frac{d}{m} + \cdots + \frac{m-d}{m}) = \frac{m-d}{2}.
$$
则所求
$$
\sum_{0 \leq k < m} \lfloor \frac{nk+x}{m} \rfloor = d \lfloor \frac{x}{d} \rfloor + \frac{m-1}{2}n + \frac{d-m}{2}.
$$
