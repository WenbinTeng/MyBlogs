---
title: concrete-math-numbers-mod
date: 2024-04-18 19:26:57
tags:
---

# 模 m 剩余类 Residue Class of Module m

我们把与整数 $a$ 模 $m$ 同余的整数称作同余数，这些同余数构成的集合记作 $\bar{a}=\set{x \in Z | x \equiv a \pmod m}$。所有模 $m$ 的同余数集合所构成的集合称为模 $m$ 的剩余类，记作 $z_m = \set{\overline{1},\overline{2},\dots,\overline{m-1}}$。模 $m$ 剩余类具有如下性质，假设 $a \equiv b \pmod m, c \equiv d \pmod m$。
$$
a + b \equiv c + d \pmod m; \\
ab \equiv cd \pmod m.
$$
我们定义以下符号作为剩余类的集合运算符号。
$$
\bar{a} + \bar{b} \overset{def}{=} \overline{a+b}; \\
\bar{a} \cdot \bar{b} \overset{def}{=} \overline{a \cdot b}.
$$
剩余类的加法和乘法具有以下性质。

- 封闭性：加法和乘法都具有封闭性，有限个元素的运算其结果也是有限个元素。
- 零元与单位元：加法具有零元，即 $\forall \bar{a} + \bar{0} = \bar{a}; \forall \bar{0} + \bar{a} = \bar{a}$；乘法<u>可能</u>具有单位元，即 $\forall \bar{a} \cdot \bar{e} = \bar{a}; \forall \bar{e} \cdot \bar{a} = \bar{a}$。
- 逆元：加法具有可逆性，即 $\forall \bar{a}, \exist b \ s.t. \bar{a} + \bar{b} = \bar{0}$；乘法<u>可能</u>具有可逆性，即 $ \forall \bar{a}, \exist b \ s.t. \bar{a} \cdot \bar{b} = \bar{e}$。 
- 结合律：加法和乘法都符合结合律。
- 交换律：加法符合交换律，乘法<u>可能</u>符合交换律。
- 分配律：加法和乘法都符合分配律。

根据群环域的概念，我们对模 $m$ 剩余类环进行分类，以在模 $m$ 剩余类上模拟数字的运算。

- 若 $m$ 为和数，那么模 $m$ 剩余类具有零元，当 $a \pmod m = 1$时存在逆元。
- 若 $m$ 为素数，那么模 $m$ 剩余类构成域。
