---
title: programming-test-25-05-15
date: 2025-05-15 00:03:32
tags:
---

# 0515 手撕记录

[11. 盛最多水的容器 - 力扣（LeetCode）](https://leetcode.cn/problems/container-with-most-water/)

给定一个长度为 `n` 的整数数组 `height` 。有 `n` 条垂线，第 `i` 条线的两个端点是 `(i, 0)` 和 `(i, height[i])` 。

找出其中的两条线，使得它们与 `x` 轴共同构成的容器可以容纳最多的水。

返回容器可以储存的最大水量。

**说明：**你不能倾斜容器。



**示例 1：**

![img](https://aliyun-lc-upload.oss-cn-hangzhou.aliyuncs.com/aliyun-lc-upload/uploads/2018/07/25/question_11.jpg)

```
输入：[1,8,6,2,5,4,8,3,7]
输出：49 
解释：图中垂直线代表输入数组 [1,8,6,2,5,4,8,3,7]。在此情况下，容器能够容纳水（表示为蓝色部分）的最大值为 49。
```

**示例 2：**

```
输入：height = [1,1]
输出：1
```

 

**提示：**

- `n == height.length`
- `2 <= n <= 105`
- `0 <= height[i] <= 104`



##### 思路：双指针

我们使用双指针维护当前的盛水容器。初始时，两个指针分别指向数组的左右两端。其中一个指针在比较后向中间移动，更新容器的储水最大值。每次都选择指向较小高度的指针向中间移动，因为每次更新都会使容器的宽度变小，因此需要选择更高的边界，以确保能选择最优的结果。



##### 代码

```cpp
#include <iostream>
#inlcude <vector>

int main() {
    int n;
    std::cin >> n;

    std::vector<int> height(n);
    for (int i = 0; i < n; i++) {
        std::cin >> height[i];
    }

    auto getArea = [&](int left, int right) -> int {
        return std::min(height[left], height[right]) * (right - left);
    };

    int maxArea = 0;
    int left = 0;
    int right = n - 1;
    while (left < right) {
        maxArea = std::max(maxArea, getArea(left, right));

        if (height[left] < height[right]) {
            left++;
        } else {
            right--;
        }
    }

    std::cout << maxArea;

    return 0;
}
```

