---
title: data-structure-binary-indexed-tree
date: 2024-12-09 14:54:41
tags:
---

# 树状数组 Binary Indexed Tree

在算法竞赛和实际应用中，处理动态数据时经常遇到以下问题：

- **区间求和**：快速求解某个区间的元素总和；
- **单点更新**：快速更新某个元素的值，同时保证区间查询结果的正确性。

如果使用简单的数组实现，区间查询的复杂度为 $O(n)$，单点更新的复杂度为 $O(1)$。而如果选用前缀和数组，查询变成了 $O(1)$，但更新需要 $O(n)$。如何同时兼顾高效的查询与更新呢？**树状数组**，也成为二叉索引树（Binary Indexed Tree）提供了优雅的解决方案，其查询与更新的时间复杂度均为 $O(\log{n})$。本文将深入讲解树状数组的原理、实现与应用。



### 1. 什么是树状数组

树状数组的核心是通过数组下标的二进制特性，将数据划分为不同的区间。每个数组元素存储一个子区间的累加值，利用这些部分和快速计算前缀和或更新元素。其特性如下：

- 树状数组的本质是一棵隐式的二叉索引树，存储在一个一维数组中；
- 下标 $i$ 的元素存储的是区间 $[i-2^r+1,i]$ 的和，其中 $r$ 是 $i$ 的二进制表示中最低位 1 的位置。例如：
  - $i=4$（二进制为 $100$），维护区间 $[1,4]$；
  - $i=6$（二进制为 $110$），维护区间 $[5,6]$。



### 2. 树状数组的基本操作

**2.1 单点更新**

更新某个下标元素值时，需要修改受影响的所有区间值。更新规则是：
$$
i \rightarrow i + \mathrm{lowbit}(i)
$$
其中，$\mathrm{lowbit}(i)$ 表示 $i$ 的二进制中最低位 1 所代表的值，可以通过位运算求得：
$$
\mathrm{lowbit}(i) = i \& (-i)
$$

```c
int lowbit(int x) {
	return x & (-x);
}

void update(int index, int delta) {
    while (index <= tree.size()) {
        tree[index] += delta;
        idx += lowbit(index);
    }
}
```



**2.2 前缀和查询**

查询数组前 $i$ 个元素的前缀和时，只需累加相关区间的值即可。查询规则为：
$$
i \rightarrow i - \mathrm{lowbit}(i)
$$

```c
int query(int index) {
    int sum = 0;
    while (index > 0) {
        sum += tree[index];
        index -= lowbit(index);
    }
    return sum;
}
```

如果需要查询的是区间 $[l,r]$ 内元素的和，那么可以转化为对区间 $[1,r]$ 和区间 $[1,l-1]$ 前缀和的查询再做差分。



### 3. 树状数组的复杂度分析

- 时间复杂度：修改和查询操作的时间复杂度都是 $O(\log{n})$。
- 空间复杂度：$O(n)$。



### 4. 树状数组的模板

```c++
class BinaryIndexedTree {
public:
    BinaryIndexedTree(int n) {
        tree.resize(n + 1, 0);
    }

    void update(int index, int delta) {
        while (index < tree.size()) {
            tree[index] += delta;
            index += lowbit(index);
        }
    }

    int query(int index) {
        int sum = 0;
        while (index > 0) {
            sum += tree[index];
            index -= lowbit(index);
        }
        return sum;
    }

    int rangeQuery(int left, int right) {
        return query(right) - query(left - 1);
    }

private:
    std::vector<int> tree;
    
    int lowbit(int x) {
        return x & (-x);
    }
};
```



### 5. 实例

[307. 区域和检索 - 数组可修改 - 力扣（LeetCode）](https://leetcode.cn/problems/range-sum-query-mutable/description/)

[308. 二维区域和检索 - 矩阵可修改 - 力扣（LeetCode）](https://leetcode.cn/problems/range-sum-query-2d-mutable/description/)
