---
title: data-structure-monotonic-stack
date: 2024-11-22 09:59:40
tags:
---

# 单调栈 Monotonic Stack

在算法问题中，许多问题都与**单调性**相关。例如：

- 寻找数组中某个元素左侧或右侧比它大的元素；
- 解决栈相关的问题，比如计算有效的括号、接雨水问题；
- 处理与区间、滑动窗口相关的动态问题。

许多这样的问题如果直接暴力处理，可能会导致 $O(n^2)$ 的复杂度，这在面对大数据时会变得非常低效。为了提高效率，我们可以使用一种叫做**单调栈（Monotonic Stack）**的数据结构，它能将许多问题的复杂度优化到 $O(n)$，并且简单高效。

本文将介绍单调栈的基本概念、操作方法、应用场景以及常见题目，帮助你在遇到类似问题时能够游刃有余。



### 1. 什么是单调栈

单调栈是一个**栈**结构，它的元素满足一定的单调性（递增或递减）。简单来说，单调栈的核心思想是：

- **单调递增栈：** 栈内的元素按递增的顺序排列，栈顶元素是当前栈中最小的元素。
- **单调递减栈：** 栈内的元素按递减的顺序排列，栈顶元素是当前栈中最大的元素。

通过维护这种单调性，我们可以高效地解决许多和“下一个更大元素”或“下一个更小元素”相关的问题。



### 2. 单调栈的基本操作

单调栈的入栈与出栈操作需要根据当前元素与栈顶元素的大小关系来决定是否出栈，直到栈内元素符合单调性的需求。

- **单调递增栈：**单调递增栈确保栈内元素从底到顶是递增的。当遇到一个新的元素时，栈顶比当前元素小的元素会出栈，直到栈顶元素大于当前元素为止。这样栈中始终保持递增的顺序。

  ```c++
  std::stack<int> s;
  void insert(int x) {
      while (!s.empty() && s.top() < x)
          s.pop();
      s.push(x);
  }
  ```

- **单调递减栈：**单调递减栈则相反，栈内元素保持递减顺序。当遇到一个新的元素时，栈顶比当前元素大的元素会出栈，直到栈顶元素小于当前元素为止。这样栈中始终保持递减的顺序。



### 3. 单调栈的复杂度分析

- 时间复杂度：遍历 $n$ 个元素的数组时，每个元素至多入栈一次和出栈一次，因此处理数组的时间复杂度为 $O(n)$。
- 空间复杂度：$O(n)$。



### 4. 单调栈的应用

**4.1 下一个更大的元素**

给定一个数组 `arr`，要求找出每个元素右边的第一个比它大的元素。若没有这样的元素，则输出 -1。

**解决思路：**

- 使用单调递减栈，从右到左遍历数组。
- 当栈顶元素小于当前元素时，栈顶元素出栈，直到栈顶元素大于当前元素或栈为空。
- 当前元素的下一个更大元素即为栈顶元素（如果栈为空则为 -1）。

```c++
std::vector<int> nextGreaterElement(std::vector<int>& arr) {
    int n = arr.size();
    std::vector<int> result(n, -1);
    std::stack<int> s;

    for (int i = n - 1; i >= 0; i--) {
        while (!s.empty() && s.top() <= arr[i]) {
            s.pop();
        }
        if (!s.empty()) {
            result[i] = s.top();
        }
        s.push(arr[i]);
    }

    return result;
}
```

**4.2 接雨水问题**

给定一个非负整数数组 `height`，每个元素代表一个柱子的高度。求在这些柱子之间可以接多少雨水。

**解题思路：**

每个位置上的水量是由其左侧和右侧的最大高度决定的，具体计算可以通过单调栈来实现。

- 使用单调递减栈，从左往右遍历数组，计算每个元素的接雨水量。
- 对于每个元素，栈顶元素之前的所有柱子都可能作为容器的边界，当遇到一个更高的柱子时，就可以计算这个区间的水量。

```c++
int trap(std::vector<int>& height) {
    int n = height.size();
    int water = 0;
    std::stack<int> s;

    for (int i = 0; i < n; ++i) {
        while (!s.empty() && height[i] > height[s.top()]) {
            int top = s.top();
            s.pop();

            if (st.empty())
                break;

            int distance = i - s.top() - 1;
            int bounded_height = std::min(height[i], height[s.top()]) - height[top];
            water += distance * bounded_height;
        }
        st.push(i);
    }

    return water;
}
```



### 5. 实例

[42. 接雨水 - 力扣（LeetCode）](https://leetcode.cn/problems/trapping-rain-water/description/)

[84. 柱状图中最大的矩形 - 力扣（LeetCode）](https://leetcode.cn/problems/largest-rectangle-in-histogram/description/)

[85. 最大矩形 - 力扣（LeetCode）](https://leetcode.cn/problems/maximal-rectangle/description/)

[496. 下一个更大元素 I - 力扣（LeetCode）](https://leetcode.cn/problems/next-greater-element-i/description/)

[503. 下一个更大元素 II - 力扣（LeetCode）](https://leetcode.cn/problems/next-greater-element-ii/description/)

[739. 每日温度 - 力扣（LeetCode）](https://leetcode.cn/problems/daily-temperatures/description/)
