---
title: data-structure-cantor-encoding
date: 2025-04-13 11:02:28
tags:
---

# 康托编码 Cantor Encoding

我们常常会遇到一个问题：如何标识一个全排列中的一个排列？**康托编码**（Cantor Encoding）是一种将排列与整数进行双向映射的技巧，是康托展开在排列问题中的应用，可以高效完成状态压缩、快速查找、还原排列等操作。



### 1. 康托编码的定义

给定一个长度为 $n$ 的排列  $P = [p_0, p_1, \dots, p_{n-1}]$，其中每个 $p_i$ 是 $0 \sim n-1$ 中不重复的整数，康托编码定义如下：
$$
C(P) = \sum_{i=0}^{n-1}a_i \cdot (n-1-i)!
$$
其中，$a_i$ 表示在 $p_i$ 之后还剩多少个比 $p_i$ 小的数字没有出现过。举一个例子，求排列 $P=[2,0,3,1]$ 在全排列中的位置：

- 初始化未使用的数字集合：$\{0,1,2,3\}$。
- 第 0 位是数字 2，它在集合 $\{0,1,2,3\}$ 中是第 3 小，即 $a_0 = 2$，贡献 $c = 2 \times 3! = 12$。删除 2，剩余 $\{0,1,3\}$。
- 第 1 位是数字 0，它在集合 $\{0,1,3\}$ 中是第 1 小，即 $a_1 = 0$，贡献 $c = 0 \times 2! = 0$。删除 0，剩余 $\{1,3\}$。
- 第 2 位是数字 3， 它在集合 $\{1,3\}$ 中是第 2 小，即 $a_2 = 1$，贡献 $c = 1 \times 1! = 1$。删除 3，剩余 $\{1\}$。
- 第 3 位是数字 1，它在集合 $\{1\}$ 中是第 1 小，即 $a_3 = 0$，贡献 $c = 0 \times 0!$。删除 1，剩余 $\{\}$。
- 计算贡献共和 $C(P) = 12 + 0 + 1 + 0 = 13$。

因此，排列 $P=[2,0,1,3]$ 的康托编码为 13，即在全排列中的位置是 13。



### 2. 康托编码的计算

##### 2.1 编码

```c++
int cantorEncode(std::vector<int>& perm) {
    int n = perm.size();
    int res = 0;
    std::vector<bool> used(n, false);
    
    for (int i = 0; i < n; i++) {
        int count = 0;
        for (int j = 0; j < perm[i]; j++) {
            if (!used[j])
                count++;
        }
        res += count * std::tgamma(n - i);
        used[perm[i]] = true;
    }
    
    return res;
}
```

##### 2.2 解码

```c++
std::vector<int> cantorDecode(int k, int n) {
    std::vector<int> nums;
    for (int i = 0; i < n; i++)
    	nums.push_back(i);
    
    std::vector<int> res;
    
    for (int i = n; i >= 1; i--) {
        int factorial = tgamma(i);
        int index = k / factorial;
        code %= factorial;
        res.push_back(nums[index]);
        nums.erase(nums.begin() + index);
    }
    
    return res;
}
```



### 3. 处理重复元素

如果某个元素有重复，我们需要除以它们重复出现次数的阶乘，这是因为重复元素的排列方式并不会产生新的独特排列。在计算每个元素排名时，使用它们出现的次数来调整其排名。例如，如果一个元素已经出现过，那么它的“排名”会受到前面已经出现过相同元素的影响。我们可以通过统计每个元素的剩余数量来正确计算它的位置。

##### 3.1 编码

```C++
int cantorEncode(std::vector<int>& perm) {
    int n = perm.size();
    int res = 0;
    std::vector<bool> used(n, false);
    
    std::map<int, int> freq;
    for (auto x : perm)
        freq[x]++;
    
    for (int i = 0; i < n; i++) {
        int count = 0;
        for (auto& [x, c] : freq) {
            if (x < perm[i])
                count += c;
		}
        res += count * std::tgamma(n - i);
        freq[perm[i]]--;
        if (freq[perm[i]] == 0)
            freq.erase(perm[i]);
    }
    
    return res;
}
```

##### 3.2 解码

```c++
std::vector<int> cantorDecode(int k, std::vector<int>& perm) {
    int n = perm.size();
    std::map<int, int> freq;
    for (auto x : perm)
        freq[x]++;
    
    std::vector<int> res;
    
    for (int i = 0; i < n; i++) {
        int factorial = tgamma(i);
        int index = k / factorial;
        code %= factorial;
        int count = -1;
        for (auto& [x, c] : freq) {
        	if (count == index) {
                res.push_back(x);
                freq[x]--;
                if (freq[x] == 0)
                    freq.erase(x);
            	break;
            }
            count += c;
		}
    }
    
    return res;
}
```



### 4. 注意

- 请注意 `int` 类型的大小能否存储全排列编码和阶乘；
- 使用迭代或查表代替 `gamma` 函数计算阶乘能降低时间复杂度。
