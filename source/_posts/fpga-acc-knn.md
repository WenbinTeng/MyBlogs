---
title: fpga-acc-knn
date: 2024-11-29 14:56:36
tags:
---

# K 邻近算法加速

K近邻算法（K-Nearest Neighbors，简称KNN）是机器学习中最基础的分类与回归算法之一，它利用邻近度来对单个数据点的分组进行分类或预测。KNN 在许多实际应用中表现良好，特别是在小规模数据集上。本文将详细介绍 KNN 的原理、实现步骤及其应用场景，并通过示例代码演示其工作过程。



### 1. KNN 算法流程

K 近邻算法是一种基于实例学习的方法，用于分类或回归问题。其核心步骤是：

1. **计算距离**：计算训练样本和测试样本中每个样本点的距离（可以选择不同的距离度量）。
2. **确定邻居**：根据计算的距离进行排序，选择距离最近的 K 个样本作为邻居。
3. 分类或回归
   - **分类**：统计 K 个邻居的标签，通过多数表决法确定预测的类别；
   - **回归**：对 K 个邻居的目标值进行平均或者加权平均，作为预测值。

在 KNN 中可以应用多种距离度量，以下是常见的距离度量。

**欧式距离**：$d(x,y) = \sqrt{\sum_{i=1}^n(x_i-y_i)^2}$

**曼哈顿距离**：$d(x,y) = \sum_{i=1}^n|x_i-y_i|$

**闵科夫斯基距离**：$d(x,y) = \sqrt[p]{\sum_{i=1}^n|x_i-y_i|}$

**汉明距离**：$d(x,y) = \sum_{i=1}^k|x_i \oplus y_i|$



### 2. FPGA 加速 KNN 算法

下面我们对 rosetta 基准测试 [1] 进行修改，在 FPGA 上使用 KNN 算法执行数字识别任务。

**2.1 计算距离**

这里使用汉明距离作为度量，在 FPGA 上对样本的每一个二进制位进行异或操作并累加即可实现距离计算。

```c++
// popcount function
int popcount(WholeDigitType x) {
    // most straightforward implementation
    // actually not bad on FPGA
    int cnt = 0;
    for (int i = 0; i < 256; i++)
#pragma HLS unroll
        cnt = cnt + x[i];

    return cnt;
}

// Given the test instance and a (new) training instance, this
// function maintains/updates an array of K minimum
// distances per training set.
void update_knn(WholeDigitType test_inst,
                WholeDigitType train_inst,
                int min_distances[K_CONST]) {
#pragma HLS inline

    // Compute the difference using XOR
    WholeDigitType diff = test_inst ^ train_inst;

    int dist = 0;

    dist = popcount(diff);

    int max_dist = 0;
    int max_dist_id = K_CONST + 1;
    int k = 0;

    // Find the max distance
    FIND_MAX_DIST:for (int k = 0; k < K_CONST; k++) {
        if (min_distances[k] > max_dist) {
            max_dist = min_distances[k];
            max_dist_id = k;
        }
    }

    // Replace the entry with the max distance
    if (dist < max_dist)
        min_distances[max_dist_id] = dist;

    return;
}
```

**2.2 更新邻居**

对于较大的 K 值，建议使用比较树的形式更新邻居；而对于较小的 K 值，采用顺序比较的方式更便捷。

```c++
// Given the test instance and a (new) training instance, this
// function maintains/updates an array of K minimum
// distances per training set.
void update_knn(WholeDigitType test_inst,
                WholeDigitType train_inst,
                int min_distances[K_CONST]) {
#pragma HLS inline

    // Compute the difference using XOR
    WholeDigitType diff = test_inst ^ train_inst;

    int dist = 0;

    dist = popcount(diff);

    int max_dist = 0;
    int max_dist_id = K_CONST + 1;
    int k = 0;

    // Find the max distance
    FIND_MAX_DIST:for (int k = 0; k < K_CONST; k++) {
        if (min_distances[k] > max_dist) {
            max_dist = min_distances[k];
            max_dist_id = k;
        }
    }

    // Replace the entry with the max distance
    if (dist < max_dist)
        min_distances[max_dist_id] = dist;

    return;
}
```

**2.3 投票**

由于在 FPGA 上使用多个处理单元（PE）并行计算来加速，因此需要从多个 PE 选出的多个 K 邻居集合中选出最后的 K 个最近邻居。

```c++
void knn_vote_small(int knn_set[NUM_LANE * K_CONST],
                    int min_distance_list[K_CONST],
                    int label_list[K_CONST],
                    int label_in) {
#pragma HLS inline
#pragma HLS array_partition variable = knn_set complete dim = 0
// final K nearest neighbors
#pragma HLS array_partition variable = min_distance_list complete dim = 0
// labels for the K nearest neighbors
#pragma HLS array_partition variable = label_list complete dim = 0

    int pos = 1000;

    LANES:for (int i = 0; i < NUM_LANE; i++) {
        INSERTION_SORT_OUTER:for (int j = 0; j < K_CONST; j++) {
#pragma HLS pipeline
            pos = 1000;
            INSERTION_SORT_INNER:for (int r = 0; r < K_CONST; r++) {
#pragma HLS unroll
                pos = ((knn_set[i * K_CONST + j] < min_distance_list[r]) && (pos > K_CONST)) ? r : pos;
            }

            INSERT:for (int r = K_CONST; r > 0; r--) {
#pragma HLS unroll
                if (r - 1 > pos) {
                    min_distance_list[r - 1] = min_distance_list[r - 2];
                    label_list[r - 1] = label_list[r - 2];
                } else if (r - 1 == pos) {
                    min_distance_list[r - 1] = knn_set[i * K_CONST + j];
                    label_list[r - 1] = label_in;
                }
            }
        }
    }
}

void knn_vote_final(hls::stream<int> &knn_set_stream,
                    hls::stream<LabelType> &result_stream) {

    int min_distance_list[K_CONST];
#pragma HLS array_partition variable = min_distance_list complete dim = 0

    int label_list[K_CONST];
#pragma HLS array_partition variable = label_list complete dim = 0

    int vote_list[10];
#pragma HLS array_partition variable = vote_list complete dim = 0

    for (int t = 0; t < NUM_TEST; t++) {
        INIT_0:for (int i = 0; i < K_CONST; i++) {
            min_distance_list[i] = knn_set_stream.read();
        }
        INIT_1:for (int i = 0; i < K_CONST; i++) {
            label_list[i] = knn_set_stream.read();
        }
        INIT_2:for (int i = 0; i < 10; i++) {
#pragma HLS unroll
            vote_list[i] = 0;
        }

        INCREMENT:for (int i = 0; i < K_CONST; i++) {
#pragma HLS pipeline
            vote_list[label_list[i]] += 1;
        }

        LabelType max_vote;
        max_vote = 0;

        VOTE:for (int i = 0; i < 10; i++) {
#pragma HLS unroll
            if (vote_list[i] >= vote_list[max_vote]) {
                max_vote = i;
            }
        }

        result_stream.write(max_vote);
    }
}
```

**2.4 并行计算设计**

我们采用组内并行、组间流水的方式构建单个样本的 K 近邻，在流水线末端根据 K 近邻投票产生样本对应的标签。完整代码详见 [Github]([Terris/app/digit-recognition at main · WenbinTeng/Terris](https://github.com/WenbinTeng/Terris/tree/main/app/digit-recognition))。

```
             PE   #0                 PE   #1
            ----------              ---------- 
test_data  | lane #0  | test_data  | lane #0  |
---------> | lane #1  | ---------> | lane #1  | --> ...
           | ...      |       knn  | ...      |              ------
           | lane #m  | ---------> | lane #m  | --> ... --> | vote | --> label
            ----------              ----------               ------
                 ^                       ^
train_data       |                       |
-----------------+-----------------------+--------> ...
```



### 3. NN-Descent 算法

NN-Descent 是一种用于构建 K 近邻图的高效算法，旨在解决 K 近邻图构建中的效率和扩展性问题。NN-Descent 的核心思想是**邻居的邻居更可能是邻居**。通过不断探索每个点邻居的邻居，可以逐步完善每个点的 K 近邻，从而构建一个高质量的 K 近邻图。如果使用暴力方法构建 K 近邻图的时间复杂度为 $O(n^2)$，而使用 NN-Descent 算法优化的时间复杂度可以达到 $O(n^{1.14})$。

算法的基本步骤如下：

1. **初始化**：随机为每个点选择 K 个邻居，构建初始的近似 K 近邻图。
2. **迭代优化**：对于每个点 v，计算其邻居的邻居集合 B[v]。 在 B[v] 中寻找新的 K 近邻，并更新 v 的 K 近邻列表。 重复上述过程，直到 K 近邻图不再更新。

```python
def nn_descent(data, k, max_iter=10):
    # 初始化K近邻图
    knn_graph = initialize_knn_graph(data, k)

    for _ in range(max_iter):
    	updated = False

        for v in data:
            # 计算邻居的邻居集合
            neighbors = get_neighbors(knn_graph, v)
            candidate_neighbors = set()
            for neighbor in neighbors:
                candidate_neighbors.update(get_neighbors(knn_graph, neighbor))

            # 更新K近邻列表
            new_neighbors = find_k_nearest_neighbors(data, v, candidate_neighbors, k)
            if update_knn_graph(knn_graph, v, new_neighbors):
                updated = True

        if not updated:
            break

    return knn_graph
```

为了进一步提高算法效率，还可以使用以下方法：

- **局部连接**：通过局部连接操作，减少冗余计算。
- **增量搜索**：通过布尔标记，减少重复比较。
- **采样**：通过邻居取样和反向邻居取样，缓解局部连接的高成本和冗余计算。
- **提前终止**：在每次迭代中统计更新次数，当更新次数小于阈值时终止。

详细操作参考论文 [2]。



### 4. 参考文献

[1] Zhou Y, Gupta U, Dai S, et al. Rosetta: A realistic high-level synthesis benchmark suite for software programmable FPGAs[C]//Proceedings of the 2018 ACM/SIGDA International Symposium on Field-Programmable Gate Arrays. 2018: 269-278.

[2] Dong W, Moses C, Li K. Efficient k-nearest neighbor graph construction for generic similarity measures[C]//Proceedings of the 20th international conference on World wide web. 2011: 577-586.
