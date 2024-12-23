---
title: data-structure-union-find
date: 2024-11-20 09:53:13
tags:
---

# 并查集 Union Find

在图论问题中，判断两个节点是否属于同一连通分量是一个高频任务，例如：

- 判断网络中两台计算机是否能够通信；
- 合并不同社交圈以形成更大的群组；
- 处理动态连通性问题，比如动态增加边、节点的图。

如果直接通过图的遍历（如 BFS 或 DFS）来判断连通性，效率可能不够理想，特别是在需要频繁查询或修改的情况下。这时，我们可以使用一种高效的数据结构——**并查集（Union-Find）**。

并查集以其简洁性和高效性成为解决动态连通性问题的强大工具。本文将详细介绍并查集的核心概念、操作方法、优化技巧以及实际应用场景。



### 1. 什么是并查集？

并查集是一种**树形结构**的数据结构，用于处理元素的**分组**和**连通性判断**问题。其核心思想是：

- 每个节点属于一个集合，用一棵树表示集合；
- 每个集合的树有一个“根节点”作为代表；
- 通过记录树的根节点，我们可以快速判断两个节点是否属于同一集合。



### 2. 并查集的两大核心操作

初始化：

```c
int parent[MAX_N];
void init(int n) {
	for (int i = 1; i <= n; i++)
		parent[i] = i;
}
```

**2.1 查找（Find）**

查找某个元素所属的集合代表（即根节点）。

- 如果两个元素的根节点相同，则它们属于同一个集合。

```c
int find(int x) {
	if (parent[x] == x)
		return x;
    return find(parent[x]);
}
```

**2.2 合并（Union）**

将两个元素所属的集合合并成一个集合。（为了避免使用 C/C++ 中的关键字 union，这里使用 merge 作为操作名称）

- 找到两个元素的根节点，然后将其中一个根节点挂到另一个根节点上。

```c
void merge(int x, int y) {
	int xRoot = find(x);
	int yRoot = find(y);
	if (xRoot != yRoot)
		parent[xRoot] = yRoot;
}
```



### 3. 并查集的优化方法

**3.1 路径压缩**

通过上述方法构造的并查集是比较低效的：如果在根节点前部或者在叶节点后部不断插入新的节点，则会形成一条长链，使得查找操作的用时变长。我们可以使用**路径压缩**方法降低这种影响。

如果我们只关心每个节点的根节点，那么我们在查询过程中，把沿途每个节点的父节点都设置为根节点即可。

```c
int find(int x) {
	if (parent[x] != x)
		parent[x] = find(parent[x]);
	return parent[x];
}
```

**3.2 按秩合并**

路径压缩能改善查找分支上的路径长度，但树型比较复杂时，合并操作可能会带来较长的路径。一个简单的启发式思想是，应该把层数较少的树添加到层数较深的树上，以避免更长的查找路径。

我们用一个数组 rank[] 来记录每个根节点对应的树的深度（秩）。初始时所有元素的 rank 值为 1。合并时比较两个根节点的 rank 值，把 rank 值较大的根节点作为 rank 值较小的根节点的前驱。

```c
void merge(int x, int y) {
	int xRoot = find(x);
	int yRoot = find(y);
    if (xRoot != yRoot) {
        if (rank[xRoot] > rank[yRoot])
            parent[yRoot] = xRoot;
        else {
            parent[xRoot] = yRoot;
            if (rank[xRoot] == rank[yRoot])
                rank[yRoot]++;
        }
    }
}
```



### 4. 并查集的复杂度分析

- 时间复杂度：同时使用路径压缩和启发式合并之后，并查集的每个操作平均时间为 $O(\alpha(n))$，其中 $\alpha(n)$ 为阿克曼函数的反函数，其增长极其缓慢，也就是说其单次操作的平均运行时间可以认为是一个很小的常数。
- 空间复杂度：$O(n)$。



### 5. 并查集的模板

```c++
class UnionFind {
public:
    void init(int n) {
        parent = std::vector<int>(n);
        rank = std::vector<int>(n, 1);
        
        for (int i = 0; i < n; i++) {
            parent[i] = i;
        }
    }
    
    int find(int x) {
        if (parent[x] != x)
            parent[x] = find(parent[x]);
        return parent[x];
    }
    
    void merge(int x, int y) {
        int xRoot = find(x);
        int yRoot = find(y);
        if (xRoot != yRoot) {
            if (rank[xRoot] > rank[yRoot])
                parent[yRoot] = xRoot;
            else {
                parent[xRoot] = yRoot;
                if (rank[xRoot] == rank[yRoot])
                    rank[yRoot]++;
            }
        }
    }
    
private:
    std::vector<int> parent;
    std::vector<int> rank;
}
```



### 6. 实例

[684. 冗余连接 - 力扣（LeetCode）](https://leetcode.cn/problems/redundant-connection/description/)

[685. 冗余连接 II - 力扣（LeetCode）](https://leetcode.cn/problems/redundant-connection-ii/description/)
