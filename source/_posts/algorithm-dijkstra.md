---
title: algorithm-dijkstra
date: 2025-04-18 22:03:47
tags:
---

# 优先队列优化的 Dijkstra 算法 Priority Queue Optimized Dijsktra Algorithm

Dijkstra 算法是解决图论最短路径问题的经典算法，本文将介绍使用优先队列优化的 Dijkstra 算法。



### 1. Dijkstra 算法的基本步骤

Dijkstra 算法适用于**非负**权重的有向或无向图，其核心思路是：

1. 初始化：将源点到自身的距离设为 0，到其他所有顶点的距离设为无穷大；
2. 重复以下操作，直到所有顶点都被访问或者剩余的未访问顶点距离均为无穷大：
   - 在未访问顶点中选取距离源点最小的顶点 $u$；
   - 标记 $u$ 为已访问；
   - 对于从 $u$ 出发的每条边 $(u, v)$，尝试松弛：如果 $dist[u] + w(u,v) < dist[v]$，则更新 $dist[v]$。

未经优化的朴素实现中，每次寻找最小距离顶点的操作需要 $O(n)$ 时间，因此总体时间复杂度为 $O(n^2 + m)$，对于稠密图尚可，对于稀疏图（$m ≪ n^2$）则效率不高。



### 2. 使用优先队列优化 Dijkstra 算法

我们可以使用优先队列优化寻找当前最短距离顶点的过程：将候选顶点放入最小堆后，每次只需要消耗 $O(\log{n})$ 时间复杂度即可取出距离最小的顶点，其核心思路是：

1. 用一个最小堆 $pq$ 存放 $(dist, vertex)$ 对，按 $dist$ 从小到大排列；
2. 初始时，向 $pq$ 中插入源点 $(0, src)$；
3. 在主循环中：
   - 从 $pq$ 中取出堆顶元素 $(d, u)$；
   - 如果 $d > dist[u]$，说明这是旧的条目，直接跳过；
   - 否则，对 $u$ 的所有邻边 $(u, v)$ 执行松弛操作，若 $dist[u] + w(u,v) < dist[v]$，更新 $dist[v]$ 并将 $(dist[v], v)$ 插入 $pq$​。

这样，每条边最多可能被插入堆中一次或两次，堆的操作代价为 $O(\log{n})$，整体时间复杂度可降至 $O((n + m) \log{n})$，有利于提升稀疏图的处理效率。



### 3. 代码实现

```c++
using namespace std;
using Edge = pair<int, int>;
using Graph = vector<vector<Edge>>;

vector<long long> dijkstra(const Graph& graph, int src) {
    const long long INF = LLONG_MAX;
    int n = graph.size();
    vector<long long> dist(n, INF);
    dist[src] = 0;

    priority_queue<pair<long long, int>, vector<pair<long long, int>>, greater<>> pq;
    pq.push({0, src});

    while (!pq.empty()) {
        auto [d, u] = pq.top();
        pq.pop();

        if (d > dist[u])
            continue;

        for (auto& [v, w] : graph[u]) {
            if (dist[u] + w < dist[v]) {
                dist[v] = dist[u] + w;
                pq.push({dist[v], v});
            }
        }
    }

    return dist;
}
```

