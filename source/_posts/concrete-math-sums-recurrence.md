---
title: concrete-math-sums-recurrence
date: 2024-03-27 19:09:23
tags:
---

# 和式与递归式 Sums and Recurrence

基于一些和式计算的特殊方法，在将递归式转换为和式后，可以简化递归式的求解。

**使用和式求解汉诺塔问题**

考虑汉诺塔问题的递归关系：
$$
\begin{align*}
T_0 &= 0; \\
T_n &= 2T_{n-1} + 1, n > 0.
\end{align*}
$$
等式两边同时除以 $2^n$，得
$$
\begin{align*}
T_0/2^0 &= 0; \\
T_n/2^n &= T_{n-1}/2^{n-1} + 1/2^{n-1}, n > 0.
\end{align*}
$$
令 $S_n = T_n/2^n$， 有
$$
\begin{align*}
S_0 &= 0; \\
S_n &= S_{n-1} + 2^{-n}, n > 0.
\end{align*}
$$
由此得到
$$
\begin{align*}
S_n = \sum_{k=1}^{n} 2^{-k} = (\frac{1}{2})^1 + (\frac{1}{2})^2 + \cdots + (\frac{1}{2})^n = 1 - (\frac{1}{2})^n.
\end{align*}
$$
因此 $T_n = 2^nS_n = 2^n-1$。上述过程的核心思想在于，利用一个**求和因子**在等式两边相乘。对于形如 $a_nT_n=b_nT_n+c_n$ 的递归关系，求和因子 $s_n$ 满足 $s_nb_n=s_{n-1}a_{n-1}$，从而得到递归式
$$
\begin{align*}
S_n &= s_na_nT_n = s_nb_nT_{n-1} + s_nc_n = S{n-1} + s_nc_n \\
&= s_0a_0T_0 + \sum_{k=1}^{n}s_kc_k \\
&= s_1b_1T_0 + \sum_{k=1}^{n}s_kc_k.
\end{align*}
$$
则 $T_n = \frac{1}{s_na_n}(s_1b_1T_0 + \sum_{k=1}^{n}s_kc_k)$。那么我们如何求出 $S_n$ 呢？我们根据 $s_n=\frac{a_{n-1}}{b_n}s_{n-1}$ 进行展开，发现求和因子 $s_n = \frac{a_{n-1}a_{n-2}\dots a_1}{b_nb_{n-1}\dots b_2}$ 或为其整数倍时容易求解，如汉诺塔问题中的 $a_n=1, b_n=2, s_n=2^{-n}$。注意该分式的分母不能包含零项。

**使用和式求解快速排序比较次数问题**

快速排序算法的平均比较次数 $C_n$ 满足递归式
$$
\begin{align*}
C_0 &= C_1 = 0; \\
C_n &= n + 1 + \frac{2}{n}\sum_{k=0}^{n-1}C_k, n > 1.
\end{align*}
$$
首先我们使用 $n$ 乘以等式两边，得
$$
nC_n = n^2 + n + 2\sum_{k=0}^{n-1}C_k.
$$
然后我们使用 $n-1$ 替换上式中的 $n$，得
$$
(n-1)C_{n-1} = (n-1)^2+(n-1)+2\sum_{k=0}^{n-2}C_k.
$$
最后两式相减以消除和式，有
$$
nC_n - (n-1)C_{n-1} = n^2 + n + 2C_{n-1} - (n-1)^2 - (n-1) = 2n + 2C_{n-1}.
$$
我们得到化简后的递归式
$$
\begin{align*}
C_0 &= C_1 = 0; \\
nC_n &= (n+1)C_{n-1} + 2n, n \geq 2.
\end{align*}
$$
将前述的一般形式递归关系求解方法代入，得
$$
C_n = 2(n+1)\sum_{k=1}^{n}\frac{1}{k+1} - \frac{2}{3}(n+1) = 2(n+1)H_n - \frac{8}{3}n - \frac{2}{3}, n > 1.
$$
**使用和式求解Top-K比较次数问题**

我们可以参考快速排序的思想，在一个长度为 $n$ 的序列中选择出第 $K$ 大的元素，其比较次数为$C_n$。该算法的伪码如下所示。

```c
int partition(int a[], int start, int end) {
    int key = a[start];
    while (start < end) {
        // largers on the left.
        while (start < end && a[end] <= key)
            end--;
        if (start < end) {
            swap(a[start], a[end]);
        }
        // smallers on the right.
        while (start < end && a[start] > key)
            start++;
        if (start < end) {
            swap(a[end], a[start]);
        }
    }
    a[start] = key;
    return start;
}
int kth_qsort(int a[], int start, int end, int k) {
    int pivot = -1, len = 0;
    if (start < end) {
        pivot = partition(a, start, end);
        len = pivot - start + 1;
        if (len < k) {
            pivot = kth_qsort(a, pivot + 1, end, k - len);
        } else if (len > k) {
            pivot = kth_qsort(a, start, pivot - 1, k);
        }
    }
    return pivot;
}
int kth_top(int a[], int k) {
    int p = kth_qsort(a, 0, N, k);
    return a[p];
}
```

对于一次长度为 $n$ 的递归过程，枢纽变量需要与其他元素进行 $n$ 次比较。下一次递归过程只需要对枢纽变量一侧的序列进行处理，序列长度为 $i$ 的概率为 $\frac{1}{n}$，比较次数为 $c_i$。因此，在长度为 $n$ 的序列中找出第 $K$ 大个元素所需的总比较次数满足的递归关系为
$$
C_n = n + \frac{1}{n}\sum_{i=0}^{n-1}C_i.
$$
等式两边同乘以 $n$，得
$$
nC_n = n^2 + \sum_{i=0}^{n-1}C_i.
$$
使用 $n-1$ 代替 $n$，得
$$
(n-1)C_{n-1} = (n-1)^2 + \sum_{i=0}^{n-2}C_i.
$$
两式相减以消去和式，有
$$
nC_n - (n-1)C_{n-1} = n^2 - (n-1)^2 + C_{n-1}.
$$
化简得到递归关系
$$
\begin{align*}
C_0 &= C_1 = 0, \\
nC_n &= nC_{n-1} + 2n-1, n \geq 2.
\end{align*}
$$
直接展开
$$
\begin{align*}
C(n) &= C_{n-1} + 2 - \frac{1}{n} \\
     &= C_{n-2} + 2 - \frac{1}{n-1} + 2 - \frac{1}{n} \\
     &= \cdots \\
     &= 2(n-1) - \sum_{i=1}^{n-1}\frac{1}{i} \\
     &= 2(n-1) - H_{n-1} \\
     & \sim O(n).
\end{align*}
$$
**调和级数**

上述过程提及的级数 $H_n = \sum_{i=1}^{n}\frac{1}{i}$ 称为调和级数，其计算时间复杂度推导如下。
$$
\begin{align*}
H_n = \sum_{i=1}^{n}\frac{1}{i} &= 1 + (\frac{1}{2}+\frac{1}{3}) + (\frac{1}{4}+\frac{1}{5}+\frac{1}{6}+\frac{1}{7}) + \cdots \\
&\leq 1 + (\frac{1}{2}+\frac{1}{2}) + (\frac{1}{4}+\frac{1}{4}+\frac{1}{4}+\frac{1}{4}) \\
&= \underbrace{1 + 1 + 1 + \cdots}_{\lceil\log{n}\rceil} \\
&\sim O(\log{n})
\end{align*}
$$
