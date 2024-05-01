---
title: concrete-math-numbers-primes
date: 2024-04-17 18:49:51
tags:
---

# 素数 Primes

如果一个正整数 $p$ 恰好只有两个因子，即 $1$ 和 $p$，那么这个数就称为素数，而其他有非平凡因子的数都称为合数。以下四个关系等价。

- $p$ 是素数
- $\forall a$，有 $p \mid a$ 或 $\gcd(p,a)=1$
- $\forall a,b$ ，若 $p \mid ab$ 则 $p \mid a$ 或 $p \mid b$
- $p$ 不能分解为两个比 $p$ 小的正整数的积

**算数基本定理**

任何正整数 $n$ 都可以表示为素数的乘积，且其构成非减素数乘积序列是唯一的，即
$$
n = \prod_p p^{n_p}, \vee n_p \geq 0.
$$
**欧几里得数**

欧几里得数由一系列欧几里得数的乘积递归地定义，如下所示
$$
e_n = e_1 e_2 \cdots e_{n-1} +1, n \geq 1.
$$
其中 $e_1 = 2, e_2 = 3, e_3 = 7, e_4 = 43, \dots$。显然，一个欧几里得数不能被其他欧几里得数整除，但是欧几里得数不一定是素数，如 $e_5 = 1807 = 13 \times 139$ 是和数。但是欧几里得数是互素的（是否正确？如何证明？），即 $\gcd(e_m,e_n)=1$。

**梅森数**

形如 $2^p-1$ 的数，如果其是素数，则称其为梅森数。首先，梅森数的 $p$ 不能是和数，因为存在以下因式分解
$$
2^{k m}-1=\left(2^{m}-1\right)\left(2^{m(k-1)}+2^{m(k-2)}+\cdots+1\right).
$$
其次，如果 $p$ 是素数，那么 $2^p-1$ 也不一定是梅森数，如最小的非梅森数 $2^{11} -1 = 2047 = 23 \times 89$。

**互素**

如果两个整数 $a$ 和 $b$ 的最大公因子为 $1$，即 $\gcd(a,b) = 1$，那么称 $a$ 与 $b$ 互素，那么就有 $\exist u,v \ \mathrm{s.t.} \ ua+vb=1$。以下列出几个关于互素的重要推论

- $a \mid bc \and \gcd(a,b)=1 \Rightarrow a \mid c， a,b,c \in Z_+;$
- $a \mid c \and b \mid c \and gcd(a,b)=1 \Rightarrow ab \mid c, a,b,c \in Z_+;$
- $\gcd(a,c)=1 \and \gcd(b,c)=1 \Rightarrow \gcd(ab,c) = 1, a,b,c \in Z_+.$

**费马小定理**

$\forall a \in Z \and p \in P, \ \gcd(a,p)=1 \Rightarrow a^{p-1} \equiv 1 \pmod p$

**证明 1**：考虑模 $p$ 剩余类 $Z_p = \set{1,2,\dots,p-1}$，我们对每个元素乘以 $a$ 构造 $Z_p' = \set{a,2a,\dots,(p-1)a}$，而 $Z_p'$ 中每个元素是互不相同的，下面我们用反证法证明这一点。

假设剩余类 $Z_p'$ 中的两个元素 $a \cdot i$ 与 $a \cdot j$ 相同，即 $a \cdot i \equiv a \cdot j \pmod p, 1 \leq i,j \leq p-1$，那么 $a\cdot(i-j) \equiv 0 \pmod p$，则 $p \mid a(i-j)$， 但这与 $p \nmid i-j$ 和 $p \nmid a$ 矛盾。

所以我们得出集合 $Z_p'$ 是集合 $Z_p$ 的重新排列，则两个集合中所有元素的乘积是同余的，即 $(p-1)! \equiv a^{p-1} (p-1)! \pmod p \Rightarrow a^{p-1} \equiv 1 \pmod p$。

**证明 2**：我们使用二项式定理展开
$$
\begin{align*}
a^p &= (1+a-1)^p \\
&= 1 + \begin{pmatrix} p \\ 1 \end{pmatrix} (a-1)^1 + \cdots + \begin{pmatrix} p \\ k \end{pmatrix} (a-1)^k + \cdots + \begin{pmatrix} p \\ p-1 \end{pmatrix} (a-1)^{p-1} + (a-1)^p. \\
\end{align*}
$$
其中 $\begin{pmatrix} p \\ k \end{pmatrix} = \frac{p(p-1)\cdots(p-k+1)}{k!} \in Z$，则 $k! \mid p(p-1)\cdots(p-k+1)$。又 $\gcd(k!,p) = 1$，则 $k! \mid (p-1)\cdots(p-k+1)$，所以 $p \mid \begin{pmatrix} p \\ k \end{pmatrix}$。我们将二项式定理展开式的两边同除以 $p$，得到同余关系
$$
a^p \equiv 1 + (a-1)^p \pmod p.
$$
同理，我们可以得到 $a^p \equiv 2 + (a-2)^p \pmod p$。根据数学归纳法，我们得到 $a^p \equiv a \pmod p$，则 $p \mid a^p - a \Rightarrow p \mid a(a^{p-1} - 1)$，又 $\gcd(a,p)=1$，有 $p \mid a^{p-1} -1 \Rightarrow a^{p-1} \equiv 1 \pmod p$ 。

**证明 3**：我们使用数学归纳法进行证明：

- 显然 $1^p \equiv 1 \pmod p$ 成立。

- 假设 $a^p \equiv a \pmod p$ 成立。

- 由二项式定理有
  $$
  (a+1)^p = a^p + \begin{pmatrix} p \\ 1 \end{pmatrix} a^{p-1} + \cdots + \begin{pmatrix} p \\ p-1 \end{pmatrix} a + 1. \\
  $$
  因为 $\begin{pmatrix} p \\ k \end{pmatrix} = \frac{p(p-1)\cdots(p-k+1)}{k!}$ 对 $1 \leq k \leq p-1$ 成立，在模 $p$ 意义下 $\begin{pmatrix} p \\ 1 \end{pmatrix} \equiv \begin{pmatrix} p \\ 2 \end{pmatrix} \equiv \cdots \equiv \begin{pmatrix} p \\ p-1 \end{pmatrix} \equiv 0 \pmod p$ 成立，则 $(a+1)^p \equiv a^p + 1 \pmod p$。将 $a^p \equiv a \pmod p$ 代入得到 $(a+1)^p \equiv a + 1 \pmod p$，命题成立。

**威尔逊定理**

若 $p$ 为素数，则$(p-1)! \equiv -1 \pmod p$。

证明：由于 $p-1 \equiv -1 \pmod p$，则上述定理的一个等价形式为 $(p-2)! \equiv 1 \pmod p$ 。我们知道完全剩余系 $\set{0,1,2,\dots,p-1}$ 中所有非零元素 $a$ 都有逆元 $a^{-1}$，于是该完全剩余系中彼此互逆的元素乘积为 $\bar{1}$。注意到 $a$ 与 $a^{-1}$ 有可能相等，若 $a=a^{-1}$，则 $a^2 \equiv 1 \pmod p$，即 $0 \equiv a^2 -1 \equiv (a+1)(a-1) \pmod p$，这里 $a$ 只会有两个可能的解 $a=1$ 或 $a=p-1$，因为 $p \mid a-1$ 或 $p \mid a+1$。去除 $a$ 与 $a^{-1}$ 相等的情况，我们考虑集合 $\set{2,\dots,p-2}$，我们将非零元素与其逆元相互配对，有 $a_k a_k^{-1} \equiv 1 \pmod p$，将这些元素对相乘得到 $(p-2)! \equiv 1 \pmod p$，得证。

**素数定理**

素数具有无限个。对于不超过 $x$ 的素数个数 $\pi(x)$，我们有所谓的素数定理 $\pi(x) \sim \frac{x}{\ln x}$。

简化版证明：

我们考虑组合数 $\begin{pmatrix} 2n \\ n  \end{pmatrix}$ 的素因子分解
$$
\begin{pmatrix}
2n \\ n 
\end{pmatrix} = \prod_{p\leq2n} p^{l_p}.
$$
根据组合数的定义
$$
\begin{pmatrix}
2n \\ n 
\end{pmatrix} = \frac{(2n)!}{n!n!} = \frac{(2n)(2n-1)\cdots(n+1)}{n!}.
$$
我们可以得到如下关系
$$
n^{\pi(2n)-\pi(n)} \leq  \prod_{p\leq2n} p^{l_p} \leq (2n)^{\pi(2n)}.
$$
在 $l_p \leq \lfloor \log_{p}{2n} \rfloor$ 条件下成立。

首先我们证明素数定理的下界。从上式中我们有
$$
\prod_{p\leq2n} p^{l_p} = \begin{pmatrix}
2n \\ n 
\end{pmatrix} \leq (2n)^{\pi(2n)}.
$$
同时
$$
\begin{pmatrix}
    2n \\ n 
\end{pmatrix} = \frac{(2n)(2n-1)\cdots(n+1)}{n!}=\prod_{i=1}^{n}\frac{i+n}{i} \geq 2^n.
$$
因此
$$
\begin{align*}
    2^n &\leq (2n)^{\pi(2n)}, \\
    n &\leq \pi(2n) \log2n, \\
    \pi(2n) &\geq \frac{1}{2} \frac{2n}{\log2n}, \\
    \pi(n) &\sim \Omega(\frac{n}{\log n}).
\end{align*}
$$
接着我们证明素数定理的上界。根据前述的关系
$$
n^{\pi(2n)-\pi(n)} \leq  \prod_{p\leq2n} p^{l_p} = \begin{pmatrix}
    2n \\ n 
\end{pmatrix}.
$$
我们使用二项式定理进行缩放
$$
2^{2n} = (1+1)^{2n} = 
\begin{pmatrix}
    2n \\ 0 
\end{pmatrix} + 
\begin{pmatrix}
    2n \\ 1
\end{pmatrix} + \cdots +
\begin{pmatrix}
    2n \\ n 
\end{pmatrix} + \cdots +
\begin{pmatrix}
    2n \\ 2n 
\end{pmatrix} \geq 
\begin{pmatrix}
    2n \\ n 
\end{pmatrix}.
$$
从而
$$
\begin{align*}
    n^{\pi(2n)-\pi(n)} &\leq 2^{2n}, \\
    \log{n}\cdot\pi(2n)-\log{n}\pi(n) &\leq 2n, \\
    \log{2n}\cdot\pi(2n)-\log{n}\pi(n) &\leq \log{2}\cdot\pi(2n) + 2n \\
    &\leq \log{2}\cdot2n + 2n \\
    &\leq cn.
\end{align*}
$$
也即
$$
\begin{align*}
    \log{2n}\cdot\pi(2n) &\leq \log{n}\pi(n) + cn \\
    &= c(n+\frac{n}{2}+\frac{n}{4}+\cdots) \\
    &= c\cdot2n, \ (n \xrightarrow{} +\infty), \\
    \pi(2n) &\leq c\frac{2n}{\log{2n}}, \\
    \pi(n) &\sim O(\frac{n}{\log{n}}).
\end{align*}
$$
综上，我们可以得到 $\pi(x) \sim \frac{x}{\ln x}$。

**素数筛**

埃拉托色尼筛法是一种简单而古老的算法，用于查找任何给定极限的素数。这是求小质数最有效的方法之一。对于给定的上限 $N$，该算法通过从 $2$ 开始迭代地将素数的倍数标记为合数来工作。一旦 $2$ 的所有倍数都被标记为合数，下一个素数即 $3$ 的倍数也被标记为合数。这个过程一直持续到 $p\leq\sqrt{N}$，其中 $p$ 是质数。算法的伪码如下所示

```
Input: upper limit N
bool isnp[1,2,...,N] = 0
for int i=2 to floor(sqrt(N)) do
    if !isnp[i] then
        j=i^2
        while int j < N do
            isnp[j] = 1
            j = j + i
        end while
    end if
end for
```

根据伪代码，我们可以得到内部循环执行 $\frac{N}{p}-1$ 次，外部循环执行 $\sum_{p\leq\sqrt{N}}$ 次。因此，埃拉托色尼算法的运行时间为
$$
T(N) = \sum_{p\leq\sqrt{N}} (\frac{N}{p}-1).
$$
根据素数定理，我们可以得到 $i$ 是素数的概率为
$$
\pi(x) - \pi(x-1) \sim \frac{x}{\ln{x}} - \frac{x-1}{\ln(x-1)} \approx \frac{1}{\ln{x}}, \ x \xrightarrow{} +\infty.
$$
则
$$
\begin{align*}
    T(N) &= \sum_{p\leq\sqrt{N}} (\frac{N}{p}-1) \\
    &= \sum_{x=2}^{\sqrt{N}} \frac{N}{x} \times \frac{1}{\ln x} - \pi(N) \\
    &= N \sum_{x=2}^{\sqrt{N}} \frac{1}{x\ln x} - \pi(N) \\
    &\approx N \int_{2}^{\sqrt{N}} \frac{1}{x\ln x} \mathrm{d}x  - \pi(N) \\
    &= N(\ln{\ln{\sqrt{N}}} - \ln{\ln{2}}) - \pi(N) \\
    &\sim N(\ln\frac{1}{2}{\ln{N}} - \ln{\ln{2}}) - \frac{N}{\ln N} \\
    &\sim O(N\ln{\ln{N}})
\end{align*}
$$



**Stern–Brocot 树**

Stern–Brocot 树是一种用以构造满足 $m$ 与 $n$ 互素的全部非负分数 $\frac{m}{n}$ 组成的树状集合。其思想是从两个分数 $(\frac{0}{1},\frac{1}{0})$ 出发，然后重复地在两个相邻接的分数 $\frac{m}{n}$ 与 $\frac{m'}{n'}$ 之间插入 $\frac{m + m'}{n + n'}$。

| $\frac{0}{1}$ |               |               |               |               |               |               |               |               |               |               |               |               |               |               |               |               |               | $\frac{1}{0}$ |
| ------------- | ------------- | ------------- | ------------- | ------------- | ------------- | ------------- | ------------- | ------------- | ------------- | ------------- | ------------- | ------------- | ------------- | ------------- | ------------- | ------------- | ------------- | ------------- |
|               |               |               |               |               |               |               |               |               | $\frac{1}{1}$ |               |               |               |               |               |               |               |               |               |
|               |               |               |               | $\frac{1}{2}$ |               |               |               |               |               |               |               |               |               | $\frac{2}{1}$ |               |               |               |               |
|               |               | $\frac{1}{3}$ |               |               |               | $\frac{2}{3}$ |               |               |               |               |               | $\frac{3}{2}$ |               |               |               | $\frac{3}{1}$ |               |               |
|               | $\frac{1}{4}$ |               | $\frac{2}{5}$ |               | $\frac{3}{5}$ |               | $\frac{3}{4}$ |               |               |               | $\frac{4}{3}$ |               | $\frac{5}{3}$ |               | $\frac{5}{2}$ |               | $\frac{4}{1}$ |               |
| $\frac{1}{5}$ | $\frac{1}{7}$ | $\frac{3}{8}$ | $\frac{3}{7}$ |               | $\frac{4}{7}$ | $\frac{5}{8}$ | $\frac{5}{7}$ | $\frac{4}{5}$ |               | $\frac{5}{4}$ | $\frac{7}{5}$ | $\frac{8}{5}$ | $\frac{7}{4}$ |               | $\frac{7}{3}$ | $\frac{8}{3}$ | $\frac{7}{2}$ | $\frac{5}{1}$ |

我们可以把 Stern-Brocot 树看成一个表示有理数的数系，因为每一个正的最简分数都恰好出现一次。我们用字母 L 和 R 表示从这棵树的树根走到一个特定分数时向左下方或者右下方的分支前进，这样一串 L 和 R 就唯一确定了树中的一个位置。

这种表示自然引出两个问题：

- 假设 $m$ 与 $n$ 互素，则与 $\frac{m}{n}$ 对应的 L 和 R 字符串是什么？
- 给定一个由 L 和 R 组成的字符串，与它对应的分数是什么？

我们建立一个 $2 \times 2$ 的矩阵代表分数中的量，则向左移动和向右移动的矩阵操作为
$$
M(SL) = \begin{pmatrix}
n & n + n' \\
m & m + m'
\end{pmatrix} = \begin{pmatrix}
n & n' \\
m & m'
\end{pmatrix} \begin{pmatrix}
1 & 1 \\
0 & 1
\end{pmatrix} = M(S) \begin{pmatrix}
1 & 1 \\
0 & 1
\end{pmatrix}; \\
M(SR) = \begin{pmatrix}
n + n' & n' \\
m + m' & m'
\end{pmatrix} = \begin{pmatrix}
n & n' \\
m & m'
\end{pmatrix} \begin{pmatrix}
1 & 0 \\
1 & 1
\end{pmatrix} = M(S) \begin{pmatrix}
1 & 0 \\
1 & 1
\end{pmatrix}.
$$
而初始状态我们定义为单位矩阵 $I = \begin{pmatrix}
1 & 0 \\
0 & 1
\end{pmatrix}$。这样我们就得出了第二个问题的解答，对于第一个问题，我们可以使用二叉搜索求出 $\frac{m}{n}$ 在 Stern-Brocot 树中的位置，然后输出对应的 L 和 R 字符串，算法如下所示

```
S:=I
while m/n != f(S) do
	if m/n < f(s) then
		output(L);
		S:=SL;
	else
		output(R);
		S:=SR;
	end if
end while
```

**法里级数**

阶为 $N$ 的法里级数记为 $F_N$ ，它是介于 $0$ 和 $1$ 之间的分母不超过 $N$ 的所有最简分数组成的集合，且按照递增的次序排列。

| $F_1=\{$ | $\frac{0}{1},$ |                |                |                |                |                |                |                |                |                | $\frac{1}{1}$ | $\}$ |
| -------- | -------------- | -------------- | -------------- | -------------- | -------------- | -------------- | -------------- | -------------- | -------------- | -------------- | ------------- | ---- |
| $F_2=\{$ | $\frac{0}{1},$ |                |                |                |                | $\frac{1}{2},$ |                |                |                |                | $\frac{1}{1}$ | $\}$ |
| $F_3=\{$ | $\frac{0}{1},$ |                |                | $\frac{1}{3},$ |                | $\frac{1}{2},$ |                | $\frac{2}{3},$ |                |                | $\frac{1}{1}$ | $\}$ |
| $F_4=\{$ | $\frac{0}{1},$ |                | $\frac{1}{4},$ | $\frac{1}{3},$ |                | $\frac{1}{2},$ |                | $\frac{2}{3},$ | $\frac{3}{4},$ |                | $\frac{1}{1}$ | $\}$ |
| $F_5=\{$ | $\frac{0}{1},$ | $\frac{1}{5},$ | $\frac{1}{4},$ | $\frac{1}{3},$ | $\frac{2}{5},$ | $\frac{1}{2},$ | $\frac{3}{5},$ | $\frac{2}{3},$ | $\frac{3}{4},$ | $\frac{4}{5},$ | $\frac{1}{1}$ | $\}$ |

$F_n$ 可以从 $F_{n-1}$ 演变而来，即直接在 $F_{n-1}$ 的分母之和等于 $N$ 的相邻分数 $\frac{m}{n}$ 与 $\frac{m'}{n'}$ 之间插入 $\frac{m + m'}{N}$ 得来。

我们使用 $\Phi(x)$ 计算法里级数 $F_n$ 中的分数个数，定义为 $\Phi(x) = \sum_{1 \leq k \leq x} \varphi(x)$。同时，也可以使用递归式 $\Phi(x) = \frac{1}{2} \sum_{d \geq 1} \mu(d) \lfloor \frac{x}{d} \rfloor \lfloor 1 + \frac{x}{d} \rfloor$ 进行求值。
