---
title: programming-comparation
date: 2025-02-26 09:47:20
tags:
---

# 自定义比较函数 Custom comparison function

在 C++ 中，自定义比较函数通常有几种方法，主要通过以下几种方式来实现：

##### 方法一：使用标准库函数对象

```c++
#include <iostream>
#include <string>
#include <map>

int main() {
    // 使用标准库中预定义的比较器
    std::map<int, std::string, std::greater<int>> myMap();

    myMap[5] = "apple";
    myMap[3] = "banana";
    myMap[8] = "cherry";

    for (const auto& pair : myMap) {
        std::cout << pair.first << ": " << pair.second << std::endl;
    }

    return 0;
}
```

##### 方法二：使用自定义函数与函数指针

```c++
#include <iostream>
#include <string>
#include <map>

// 自定义比较函数
bool compareAsc(int a, int b) {
    return a < b;  // 从小到大排序
}

int main() {
    // 使用函数指针作为比较函数
    std::map<int, std::string, bool(*)(int, int)> myMap(compareAsc);

    myMap[5] = "apple";
    myMap[3] = "banana";
    myMap[8] = "cherry";

    for (const auto& pair : myMap) {
        std::cout << pair.first << ": " << pair.second << std::endl;
    }

    return 0;
}
```

##### 方法三：使用 Lambda 表达式

```c++
#include <iostream>
#include <string>
#include <map>
#include <functional>

int main() {
    // 使用 Lambda 表达式作为比较函数
    std::map<int, std::string, std::function<bool(int, int)>> myMap(
        [](int a, int b) {
            return a < b;  // 从小到大排序
        });

    myMap[5] = "apple";
    myMap[3] = "banana";
    myMap[8] = "cherry";

    for (const auto& pair : myMap) {
        std::cout << pair.first << ": " << pair.second << std::endl;
    }

    return 0;
}
```

##### 方法四：使用 `std::function` 作为比较函数

```c++
#include <iostream>
#include <string>
#include <map>
#include <functional>

int main() {
    // 使用 std::function 包装比较函数
    std::function<bool(int, int)> compare = [](int a, int b) {
        return a < b;  // 从小到大排序
    };

    // 将 std::function 作为比较器传递给 std::map
    std::map<int, std::string, std::function<bool(int, int)>> myMap(compare);

    myMap[5] = "apple";
    myMap[3] = "banana";
    myMap[8] = "cherry";

    for (const auto& pair : myMap) {
        std::cout << pair.first << ": " << pair.second << std::endl;
    }

    return 0;
}
```

##### 方法五：使用函数对象（仿函数）

```c++
#include <iostream>
#include <map>

// 自定义比较仿函数
struct CompareAsc {
    bool operator()(int a, int b) const {
        return a < b;  // 从小到大排序
    }
};

int main() {
    // 使用自定义比较仿函数
    std::map<int, std::string, CompareAsc> myMap;

    myMap[5] = "apple";
    myMap[3] = "banana";
    myMap[8] = "cherry";

    for (const auto& pair : myMap) {
        std::cout << pair.first << ": " << pair.second << std::endl;
    }

    return 0;
}
```



下面总结 C++ 中常见容器的默认比较器及其行为：

| 容器/算法                    | 默认比较器/比较方法 | 默认行为      |
| ---------------------------- | ------------------- | ------------- |
| `std::set` / `std::multiset` | `std::less<T>`      | 升序 / 非降序 |
| `std::map` / `std::multimap` | `std::less<T>`      | 升序 / 非降序 |
| `std::priority_queue`        | `std::less<T>`      | 降序          |
| `std::sort()`                | `std::less<T>()`    | 升序          |

