---
title: programming-test-25-04-23
date: 2025-05-15 00:02:59
tags:
---

# 0423 机试记录

以下是 0423 机试记录。



### 图像亮度坐标搜索

给定一张二维图像，图像中每个值表示该坐标下的亮度。现在给定一个亮度值 $m$，请返回离图像中心坐标最近的 $k$ 个亮度为 $m$ 的坐标 $(x, y)$。

提示：

1. 图像中元素的坐标范围 $x : [0, w - 1]$，$y : [0, h - 1]$；
2. 图像宽高 $w$，$h$ 均为奇数，图像中心坐标为 $(w - 1) / 2$，$(h - 1) / 2$；
3. 平面上两点之间的距离定义为 $|x1 - x2| + |y1 - y2|$；
4. 在距离相同的情况下，以 $x$ 小的点优先；当 $x$ 相同时，以 $y$ 小的点优先；
5. 题目可以保证至少存在一个亮度值为 $m$ 的点。

##### 输入描述

第一行输入为图像宽度 $w$ 和图像高度 $h$，以空格隔开，宽高范围为 $[1,2000]$。

第二行输入为给定亮度值 $m$，范围为 $[1,1000]$。

第三行输入为需要输出的亮度值为 $m$ 的坐标个数 $m$，范围为 $[1,100]$ ，且 $k \leq w * h$。

接下来共 $h$ 行，每行内为 $w$ 个亮度，以空格隔开，亮度范围为 $[1,1000]$。

##### 输出描述

按顺序输出坐标序列，坐标 $x$ 和 $y$ 以空格隔开，坐标间使用空格隔开如样例所示。

$x$，$y$ 坐标均不相同时以 $x$ 增序排列，$x$ 坐标相同时以 $y$ 增序排列。若满足条件的坐标个数不足 $k$，则以实际坐标个数输出。

##### 输入样例

```
5 5
10
3
10 2 3 4 5
1 2 3 4 10
1 2 3 10 5
1 10 3 4 5
1 2 3 4 5
```

##### 输出样例

```
3 2 1 3 4 1
```

##### 解决思路：排序

首先遍历所有点的亮度值，判断其是否等于 $m$：若是，则计算该点到中心坐标的距离，与坐标一并存储。
$$
d = | x-\frac{w-1}{2} | + | y-\frac{h-1}{2} |.
$$
然后，由于结果需要按照“$d$-$x$-$y$”优先级进行增序排序。这里我们使用标准库中的优先队列维护前 $k$ 个结果，将上述坐标信息按照距离进行降序排序，这样距离最大的坐标就会出现在队首，将被优先排出。

最后，得到的前 $k$ 个坐标即所求，倒序输出优先队列中的坐标即可。

该算法的时间复杂度为 $O((w*h)* \log{(k)})$。

##### 代码

```cpp
#include <iostream>
#include <vector>
#include <queue>

int main() {
    using coor_t = std::pair<int, int>;
    using info_t = std::pair<int, coor_t>;

    int w, h;
    int m;
    int k;
    std::cin >> w >> h >> m >> k;

    std::priority_queue<info_t> pq;
    int cx = (w - 1) / 2;
    int cy = (h - 1) / 2;

    for (int y = 0; y < h; y++) {
        for (int x = 0; x < w; x++) {
            int v;
            std::cin >> v;
            if (v == m) {
                int dist = std::abs(x - cx) + std::abs(y - cy);
                pq.push({dist, {x, y}});
            }
            if (pq.size() > k)
                pq.pop();
        }
    }

    std::vector<coor_t> res;

    while (!pq.empty()) {
        auto [dist, p] = pq.top();
        res.push_back(p);
        pq.pop();
    }

    while (!res.empty()) {
        auto [x, y] = res.back();
        std::cout << x << " " << y;
        res.pop_back();
        if (!res.empty())
            std::cout << " ";
    }
    
    return 0;
}
```



### 二叉树换装

给定一颗装满彩灯的二叉树，每个节点代表一个灯泡（红、绿、蓝），并且每个节点有一个开关，当按下某节点的开关后，以该节点为根节点的子树上所有节点的灯泡颜色都会根据当前的颜色按照“红->绿->蓝->红->…”的循环切换顺序切换一次颜色。
现在已知彩灯树各节点的初始颜色和目标颜色，请求出最少开关切换次数。

解释补充：

- “层序遍历”是指从上到下、从左到右逐层遍历二叉树的节点，并将遍历结果保存在一维数组中，如果某个节点在二又树中不存在，则在数组中使用 $0$ 表示。
- 切换开关的影响是“传递性”的，即切换一个节点的开关会影响以该节点为根节点的子树上所有节点的灯泡颜色。

##### 输入描述

第一行输入为一个整数 $n$，代表 $initial[]$ 和 $target[]$ 的数字大小。

第二行输入为 $n$ 个整数，代表 $initial[]$ 的元素值。

第三行输入为 $n$ 个整数，代表 $target[]$ 的元素值。

##### 输出描述

输出一个整数，表示最少开关切换次数。

##### 输入样例

```
7
1 2 3 1 2 3 1
3 1 2 3 1 2 1
```

##### 输出样例

```
3
```

##### 解决思路：先序遍历

题目要求最小的操作次数，那么由上至下计算每个节点转换到目标状态所需要的切换次数即可。对于一个根节点，假设其转换到目标状态所需的切换次数为 $k$，那么其子树中所有节点也会切换 $k$ 次状态，但不需要立即改变树中的值，在递归中向下传递切换次数的累计值即可。

该算法的时间复杂度为 $O(n)$。

##### 代码

```cpp
#include <iostream>
#include <vector>

int main() {
    int n;
    std::cin >> n;

    std::vector<int> initial(n);
    std::vector<int> target(n);

    for (int i = 0; i < n; i++) {
        int v;
        std::cin >> v;
        initial[i] = v - 1;
        
    }

    for (int i = 0; i < n; i++) {
        int v;
        std::cin >> v;
        target[i] = v - 1;
    }

    auto dfs = [&](auto& self, int node, int op) -> int {
        if (node >= n)
            return 0;
        if (initial[node] == -1)
            return 0;
        int real = (initial[node] + op) % 3;
        int rootOp = (target[node] - real + 3) % 3;
        int nextOp = op + rootOp;
        return rootOp + self(self, node * 2 + 1, nextOp) + self(self, node * 2 + 2, nextOp);
    };

    std::cout << dfs(dfs, 0, 0);
    
    return 0;
}
```



### 最赚钱的骑手

外卖骑手需要在给定的起始地点 $S_n$ 和总工作时间 $t$ 内，依次选择若干订单进行配送。每个订单由出发站点 $s$ 到达站点 $d$ 和配送费 $p$ 三元组表示，且 $s \neq d$。若同一对 $(s, d)$ 存在多笔订单，可将它们合并合并后耗时仍为 $1$ 单位，但收入为这批订单费用之和。每次配送无论合并与否，均耗时 $1$，且每笔订单只能配送一次。配送结束条件为工作时间用尽或当前地点无可接订单。输出骑手可获得的最大总收入及对应路径，若多条路径收入相同，则取字典序最小的路径。

##### 输入描述

第一行为空格分隔的两个整数，分别表示骑手的出发地点编号 $S_n$ 和工作总时间 $t$。

第二行为一个表示订单数量的整数 $M$。

第三行开始的 $M$ 行每行都是以空格分隔的三个整数 $s$，$d$ 和 $p$，$s$ 表示出发站点，$d$ 表示目标站点，$p$ 表示订单收入。

##### 输出描述

第一行是一个整数，表示骑手能获得的最多的收入。

第二行是空格分隔的整数数组，每个数字是一个地点编号，表示从开始到结束的配送路径。当多个路径能够获取相同的收入时，输出从前往后数字序最小的路径。

##### 输入样例

```
0 2
6
0 1 2
0 5 3
5 3 3
1 2 3
2 4 10
1 3 4
```

##### 输出样例

```
6
0 1 3
```

##### 解决思路：DFS

根据提供的出发点 $S_n$ 进行深度优先搜索，求出 $t$ 时间约束内的配送费最多的路径。

该算法的时间复杂度为 $O(M)$。

```cpp
#include <iostream>
#include <vector>
#include <map>
#include <unordered_map>
#include <algorithm>

struct pair_hash {
    template <class T1, class T2>
    std::size_t operator()(const std::pair<T1, T2>& p) const {
        auto h1 = std::hash<T1>{}(p.first);
        auto h2 = std::hash<T2>{}(p.second);
        return h1 ^ h2;
    }
};

int main() {
    int sn, t;
    int m;
    std::cin >> sn >> t >> m;

    std::unordered_map<std::pair<int, int>, int, pair_hash> order;
    for (int i = 0; i < m; i++) {
        int s, d, p;
        std::cin >> s >> d >> p;
        order[{s, d}] += p;
    }

    std::unordered_map<int, std::vector<int>> adj;
    for (auto& [sd, p] : order) {
        auto [s, d] = sd;
        adj[s].push_back(d);
    }
    for (auto& [s, vec] : adj) {
        std::sort(vec.begin(), vec.end());
    }

    int maxProfit = 0;
    std::vector<int> path = {sn};
    std::vector<int> bestPath = {};
    auto dfs = [&](auto& self, int node, int time, int profit, std::vector<int>& path) -> void {
        if (time == t || adj[node].empty()) {
            if (maxProfit < profit) {
                maxProfit = profit;
                bestPath = path;
            } else if (maxProfit == profit) {
                if (bestPath > path) {
                    bestPath = path;
                }
            }
            return;
        }

        for (auto& next : adj[node]) {
            auto key = std::make_pair(node, next);

            if (order[key] == -1)
                continue;
            int currProfit = order[key];
            order[key] = -1;

            path.push_back(next);
            self(self, next, time + 1, profit + currProfit, path);
            path.pop_back();

            order[key] = currProfit;
        }

        if (maxProfit < profit) {
            maxProfit = profit;
            bestPath = path;
        } else if (maxProfit == profit) {
            if (bestPath > path) {
                bestPath = path;
            }
        }
    };
    dfs(dfs, sn, 0, 0, path);
    
    std::cout << maxProfit << std::endl;
    for (int i = 0; i < bestPath.size(); i++) {
        if (i != 0)
            std::cout << " ";
        std::cout << bestPath[i];
    }

    return 0;
}
```

