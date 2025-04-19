---
title: programming-lambda-recurrence
date: 2025-02-25 20:05:07
tags:
---

# 匿名函数递归 Recursion of Anonymous Functions

在 C++ 中，匿名函数（通常是指 lambda 表达式）是无法直接递归调用自己的，因为匿名函数本身没有名字。然而，可以通过以下几种方法间接实现递归调用：

##### 方法一：通过 `std::function` 来递归调用

```c++
#include <iostream>
#include <functional>

int main() {
    std::function<int(int)> factorial = [&](int n) {
        if (n <= 1) return 1;
        return n * factorial(n - 1);
    };

    std::cout << factorial(5) << std::endl;  // 输出 120
    return 0;
}
```

在这个例子中，`factorial` 是一个 `std::function<int(int)>` 类型的变量，它能够存储一个递归的 lambda 表达式。通过捕获 `factorial` 本身，我们能够实现递归。

##### 方法二：使用函数指针作为捕获参数

```c++
#include <iostream>

int main() {
    auto factorial = [](int n, auto& self) -> int {
        if (n <= 1) return 1;
        return n * self(n - 1, self);
    };

    std::cout << factorial(5, factorial) << std::endl;  // 输出 120
    return 0;
}
```

在这个方法中，`factorial` 是一个 lambda 表达式，它通过捕获自身的引用 `self` 来实现递归调用。

