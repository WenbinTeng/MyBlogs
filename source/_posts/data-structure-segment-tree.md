---
title: data-structure-segment-tree
date: 2024-11-21 08:59:34
tags:
---

# 线段树 Segment Tree

在许多算法问题中，我们经常需要对数组的一部分进行操作，比如：

- 查询一个数组的某个区间的总和、最大值或最小值；
- 动态更新数组中的某些值，同时保持高效的区间查询；
- 应对动态区间修改和查询问题，例如批量加值或区间替换。

直接使用暴力遍历显然无法满足高效率的需求，特别是当数组规模较大或操作频繁时。这时，**线段树（Segment Tree）** 就成为了处理这类问题的强大工具。

线段树通过分治思想将数组分解成多个区间块，使得每次查询和更新操作都可以在对数时间内完成。本文将详细介绍线段树的基本概念、操作方法、代码实现及应用场景。



### 1. 什么是线段树

线段树是一种基于二叉树的数据结构，主要用于解决**区间查询**和**区间修改**问题。

- 每个节点表示数组的一个区间；
- 叶子节点表示数组的单个元素；
- 内部节点通过合并子节点的信息，表示更大的区间。

线段树的核心在于：通过递归划分区间，能够快速处理大范围的操作，同时保留局部信息以支持动态更新。



### 2. 线段树的基本操作

使用数组来存储线段树。对于一个数组 `a`，其线段树的根节点存储在数组的下标 `1` 处，左右子节点分别存储在 `2*i` 和 `2*i+1` 处。

**2.1 树的构造**

假设我们需要支持区间求和操作。线段树的构造递归如下：

- 如果当前区间为单个元素，则直接存储该值；
- 否则，将区间分为左右两部分，递归构造子树，并将子树的值合并存储到当前节点。

```c
int tree[MAX_L];
void init(int node, int[] a, int start, int end) {
    if (start == end) {
        tree[node] = a[start];
        return;
    }
    
    int mid = (start + end) / 2;
    int lnode = 2 * node;
    int rnode = 2 * node + 1;
    
    init(lnode, a, start, mid);
    init(rnode, a, mid + 1, end);
    
    tree[node] = tree[lnode] + tree[rnode];
}
```

**2.2 区间查询**

假设我们需要查询区间 `[L,R]` 的和：

- 如果当前区间完全在 `[L,R]` 范围内，直接返回该区间的值；
- 如果当前区间完全不在 `[L, R]` 范围内，返回 0；
- 如果这个区间的左孩子与 `[L,R]` 有交集，那么搜索左孩子；如果这个区间的右孩子与`[L,R]` 有交集，那么搜索右孩子。

```c
int query(int node , int start, int end, int L, int R) {
    if (start >= L && end <= R)
        return tree[node];
    if (start > R || end < L)
        return 0;
    
    int mid = (start + end) / 2;
    int lnode = 2 * node;
    int rnode = 2 * node + 1;
    int sum = 0;
    
    sum += query(lnode, start, mid, L, R);
    sum += query(rnode, mid + 1, end, L, R);
    
    return sum;
}
```

**2.3 单点修改**

假设我们需要修改数组的某个位置 `index` 的值为 `newValue`：

- 找到对应的叶子节点，并更新值；
- 回溯更新所有父节点的值。

```c
void update(int node, int start, int end, int index, int newValue) {
    if (start == end) {
        tree[node] = newValue;
        return;
    }
    
    int mid = (start + end) / 2;
    int lnode = node * 2;
    int rnode = node * 2 + 1;
    
    if (index <= mid)
        update(lnode, start, mid, index, newValue);
    else
        update(rnode, mid + 1, end, index, newValue);
    
    tree[node] = tree[lnode] + tree[rnode];
}
```



### 3. 线段树的复杂度分析

- 时间复杂度：对于 $n$ 个元素的数组，其构造的线段树的节点个数不超过 $n+\frac{n}{2}+\frac{n}{4}+\cdots = 2n$ 个，因此构造线段树的时间复杂度为 $O(n)$；区间查询的时间复杂度为 $O(\log{n})$，单点修改的时间复杂度为 $O(\log{n})$。
- 空间复杂度：$O(n)$。



### 4. 线段树的模板

```c++
class SegmentTree {
public:
    void init(int node, std::vector<int>& a, int start, int end) {
        if (node == 1)
            tree = std::vector<int>(a.size() * 2 + 1);
        if (start == end) {
            tree[node] = a[start];
            return;
        }

        int mid = (start + end) / 2;
        int lnode = 2 * node;
        int rnode = 2 * node + 1;

        init(lnode, a, start, mid);
        init(rnode, a, mid + 1, end);

        tree[node] = tree[lnode] + tree[rnode];
    }
    
    int query(int node , int start, int end, int L, int R) {
        if (start >= L && end <= R)
            return tree[node];
        if (start > R || end < L)
            return 0;

        int mid = (start + end) / 2;
        int lnode = 2 * node;
        int rnode = 2 * node + 1;
        int sum = 0;

        sum += query(lnode, start, mid, L, R);
        sum += query(rnode, mid + 1, end, L, R);

        return sum;
    }
    
    void update(int node, int start, int end, int index, int newValue) {
        if (start == end) {
            tree[node] = newValue;
            return;
        }

        int mid = (start + end) / 2;
        int lnode = node * 2;
        int rnode = node * 2 + 1;

        if (index <= mid)
            update(lnode, start, mid, index, newValue);
        else
            update(rnode, mid + 1, end, index, newValue);

        tree[node] = tree[lnode] + tree[rnode];
    }
    
private:
    std::vector<int> tree;
}
```



### 5. 实例

[307. 区域和检索 - 数组可修改 - 力扣（LeetCode）](https://leetcode.cn/problems/range-sum-query-mutable/description/)

[308. 二维区域和检索 - 矩阵可修改 - 力扣（LeetCode）](https://leetcode.cn/problems/range-sum-query-2d-mutable/)

