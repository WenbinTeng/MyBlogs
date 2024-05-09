---
title: combinatorial-math-binomial-hypergeometric-transformations
date: 2024-05-08 17:25:55
tags:
---

# 超几何变换 Hypergeometric Transformations

使用普法夫反射定律可以将超几何函数转换为另外的形式，常用的两种变换为
$$
\frac{1}{(1-z)^a} F\left(\left.\begin{array}{c}a, b \\c\end{array} \right\rvert\, \frac{-z}{1-z}\right) = F\left(\left.\begin{array}{c}a, b \\c-b\end{array} \right\rvert\, z\right).
$$

$$
F\left(\left.\begin{array}{c}a, -n \\c\end{array} \right\rvert\, z\right) = \frac{(a-c)^{\underline{n}}}{(-c)^{\underline{n}}} F\left(\left.\begin{array}{c}a, -n \\1-n+a-c\end{array} \right\rvert\, 1-z\right).
$$

使用微分法可以调整超几何函数中的个别参数，比如
$$
(z\frac{\mathrm{d}}{\mathrm{d}z} + a_1) F\left(\left.\begin{array}{c}a_1, \cdots, a_m \\ b_1, \cdots, b_n \end{array} \right\rvert\, z\right) = a_1 F\left(\left.\begin{array}{c}a_1 + 1, a_2, \cdots, a_m \\ b_1, \cdots, b_n \end{array} \right\rvert\, z\right).
$$

$$
(z\frac{\mathrm{d}}{\mathrm{d}z} + b_1 - 1) F\left(\left.\begin{array}{c}a_1, \cdots, a_m \\ b_1, \cdots, b_n \end{array} \right\rvert\, z\right) = (b_1 - 1) F\left(\left.\begin{array}{c}a_1, \cdots, a_m \\ b_1 - 1, \cdots, b_n \end{array} \right\rvert\, z\right).
$$

对于高斯超几何函数，其满足微分方程
$$
z(1-z)F''(z) + (c-z(a+b+1))F'(z) - abF(z) = 0.
$$
特别地，高斯恒等式
$$
F\left(\left.\begin{array}{c}2a, 2b \\a+b+\frac{1}{2}\end{array} \right\rvert\, z\right) = F\left(\left.\begin{array}{c}a, b \\a+b+\frac{1}{2}\end{array} \right\rvert\, 4z(1-z)\right)
$$
等式两边都满足微分方程
$$
z(1-z)F''(z) + (a+b+\frac{1}{2})(1-2z)F'(z) - 4abF(z) = 0.
$$
故恒等式在两边和式收敛的情况下成立。