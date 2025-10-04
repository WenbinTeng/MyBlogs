---
title: data-structure-skip-list
date: 2025-10-04 19:15:48
tags:
---

# 跳表 Skip List

跳表（Skip List）是一种用于查找的类似于链表的数据结构，是对有序链表的改进，能够在 $O(\log{n})$ 时间内完成增加、删除、查找操作。跳表相比于树堆与红黑树，其功能与性能相当，并且跳表的代码长度相较下更短。



### 1. 什么是跳表

跳表包含多个层级，每个层级包含一个有序的短链表。在跳表中进行查找，遵循以下步骤：

1. 从跳表的最高层开始查找。
2. 对于该层的有序链表，水平地逐个比较节点，直至当前节点值大于等于目标值。如果等于目标值，则返回结果；如果大于目标值，则移动至下一层进行查找。
3. 如果在第一层仍无法找到目标值，则说明对应节点不存在。

使用这种分层查找，可以跳过链表中没有必要的比较，加快查询速度。



### 2. 跳表的基本操作

跳表一般需要支持增加、删除、查找操作。

##### 2.1 节点定义

节点除了一个存储值的变量外，还包含一个数组用于存储每一层指向下一个节点的指针。

```c++
class SkiplistNode {
    int val;
    std::vector<SkiplistNode *> forward;
    SkiplistNode(int _val, int _maxLevel = MAX_LEVEL)
        : val(_val), forward(_maxLevel, nullptr) {}
};
```

##### 2.2 增加节点

假设我们新加入的节点为 `newNode`，我们计算此次节点的插入层数 `lv`，如果 `level` 小于 `lv`，则更新 `level` 的链表。在每一个需要插入节点的层中，查找节点的插入位置，即在当前层水平地逐个比较直至当前节点的下一个节点大于等于目标节点。我们用数组 `update` 保存每一层查找的最后一个节点。我们将 `newNode` 的第 `i` 层后续节点指向 `update[i]` 的下一个节点，同时更新 `update[i]` 的后续节点为 `newNode`。

```c++
void add(int num) {
    std::vector<SkiplistNode *> update(MAX_LEVEL, head);
    SkiplistNode *curr = this->head;
    for (int i = level - 1; i >= 0; i--) {
        while (curr->forward[i] && curr->forward[i]->val < num) {
            curr = curr->forward[i];
        }
        update[i] = curr;
    }
    int lv = randomLevel();
    level = std::max(level, lv);
    SkiplistNode *newNode = new SkiplistNode(num, lv);
    for (int i = 0; i < lv; i++) {
        newNode->forward[i] = update[i]->forward[i];
        update[i]->forward[i] = newNode;
    }
}
```

##### 2.3 删除节点

首先我们需要查找当前元素是否存在于跳表中。从跳表的最高层中开始查找，在当前层水平地逐个比较直至当前节点的下一个节点大于等于目标节点，然后移动至下一层进行查找，重复这个过程直至到达第一层。如果在第一层仍无法找到目标值，则说明对应节点不存在。我们用数组 `update` 保存每一层查找的最后一个节点。我们从第一层开始逐层向上删除链表中的节点，直至该层的链表中没有目标节点。

```c++
bool erase(int num) {
    std::vector<SkiplistNode *> update(MAX_LEVEL, nullptr);
    SkiplistNode *curr = this->head;
    for (int i = level - 1; i >= 0; i--) {
        while (curr->forward[i] && curr->forward[i]->val < num) {
            curr = curr->forward[i];
        }
        update[i] = curr;
    }
    curr = curr->forward[0];
    if (!curr || curr->val != num) {
        return false;
    }
    for (int i = 0; i < level; i++) {
        if (update[i]->forward[i] != curr) {
            break;
        }
        update[i]->forward[i] = curr->forward[i];
    }
    delete curr;
    while (level > 1 && head->forward[level - 1] == nullptr) {
        level--;
    }
    return true;
}
```

##### 2.4 查询节点

从跳表的最高层开始查找，在当前层水平地逐个比较直至当前节点的下一个节点大于等于目标节点，然后移动至下一层进行查找，重复这个过程直至到达第一层。

```c++
bool search(int target) {
    SkiplistNode *curr = this->head;
    for (int i = level - 1; i >= 0; i--) {
        while (curr->forward[i] && curr->forward[i]->val < target) {
            curr = curr->forward[i];
        }
    }
    curr = curr->forward[0];
    if (curr && curr->val == target) {
        return true;
    }
    return false;
}
```



### 3. 跳表的复杂度分析

- 时间复杂度：对于一个节点，其向更高层走的概率为 $p$，该层为最高层的概率为 $1-p$，设 $C(i)$ 为在一个无限长度跳表中向上走 $i$ 层的期望代价，则 $C(i) = (1-p)(1+C(i)) + p(1+C(i-1))$，解得 $C(i)=\frac{1}{p}$。设 $L(n)$ 为含有 $n$ 个节点的跳表中的最大层数，其包含的元素个数期望为 $\frac{1}{p}$，则 $\frac{1}{p}=np^{L(n)-1}$，解得 $L(n)=\log_{p}{\frac{1}{n}}$。现在我们得知，在长度为 $n$ 的跳表中，从最底层爬到第 $L(n)$ 层的期望步数存在上界 $\frac{L(n)-1}{p}$；到达第 $L(n)$ 层后，向左走的步数不会超过第 $L(n)$ 层及更高层的节点数总和，而这个总和的期望为 $\frac{1}{p}$。所以，到达第 $L(n)$ 层后，向左走的期望步数存在上界 $\frac{1}{p}$；同理，向上走的期望步数存在上界 $\frac{1}{p}$。因此，跳表查询的期望步数为 $\frac{L(n)-1}{p}+\frac{2}{p}$，即期望时间复杂度为 $O(\log{n})$。最坏情况下，每一层的链表长度等于原始链表，此时跳表的最坏时间复杂度为 $O(n)$。

- 空间复杂度：对于一个节点，其最高层数为 $i$ 的概率为 $p^{i-1}(1-p)$，则跳表的期望层数为 $\sum_{i \geq 1}ip^{i-1}(1-p)$。因为 $p$ 是常数，所以跳表的平均空间复杂度为 $O(n)$。在最坏的情况下，每一层的链表长度等于原始链表，此时跳表的最坏空间复杂度为 $O(n\log{n})$。



### 4. 跳表的模版

```c++
constexpr int MAX_LEVEL = 32;
constexpr double P_FACTOR = 0.25;

class SkiplistNode {
  public:
    int val;
    std::vector<SkiplistNode *> forward;
    SkiplistNode(int _val, int _maxLevel = MAX_LEVEL)
        : val(_val), forward(_maxLevel, nullptr) {}
};

class Skiplist {
  private:
    SkiplistNode *head;
    int level;

  public:
    Skiplist() : head(new SkiplistNode(-1)), level(0) {}

    bool search(int target) {
        SkiplistNode *curr = this->head;
        for (int i = level - 1; i >= 0; i--) {
            while (curr->forward[i] && curr->forward[i]->val < target) {
                curr = curr->forward[i];
            }
        }
        curr = curr->forward[0];
        if (curr && curr->val == target) {
            return true;
        }
        return false;
    }

    void add(int num) {
        std::vector<SkiplistNode *> update(MAX_LEVEL, head);
        SkiplistNode *curr = this->head;
        for (int i = level - 1; i >= 0; i--) {
            while (curr->forward[i] && curr->forward[i]->val < num) {
                curr = curr->forward[i];
            }
            update[i] = curr;
        }
        int lv = randomLevel();
        level = std::max(level, lv);
        SkiplistNode *newNode = new SkiplistNode(num, lv);
        for (int i = 0; i < lv; i++) {
            newNode->forward[i] = update[i]->forward[i];
            update[i]->forward[i] = newNode;
        }
    }

    bool erase(int num) {
        std::vector<SkiplistNode *> update(MAX_LEVEL, nullptr);
        SkiplistNode *curr = this->head;
        for (int i = level - 1; i >= 0; i--) {
            while (curr->forward[i] && curr->forward[i]->val < num) {
                curr = curr->forward[i];
            }
            update[i] = curr;
        }
        curr = curr->forward[0];
        if (!curr || curr->val != num) {
            return false;
        }
        for (int i = 0; i < level; i++) {
            if (update[i]->forward[i] != curr) {
                break;
            }
            update[i]->forward[i] = curr->forward[i];
        }
        delete curr;
        while (level > 1 && head->forward[level - 1] == nullptr) {
            level--;
        }
        return true;
    }

    int randomLevel() {
        int lv = 1;
        while (random() < P_FACTOR && lv < MAX_LEVEL) {
            lv++;
        }
        return lv;
    }
};
```



### 5. 实例

[1206. 设计跳表](https://leetcode.cn/problems/design-skiplist/)
