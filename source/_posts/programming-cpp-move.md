---
title: programming-cpp-move
date: 2025-10-08 09:49:10
tags:
---

# C++ 移动语义 C++ Move

C++11 引入了**移动语义**，它的出现主要是为了解决对象频繁拷贝带来的性能开销问题。移动语义的核心思想是：当对象的资源不再需要时，可以“转移”它的内部资源，而不是进行昂贵的深拷贝。



### 背景

在传统 C++ 中，函数参数传递或者返回值通常依赖拷贝构造函数。比如，在调用函数前，通常需要在调用者的作用域中创建一个（右值）对象，然后作为参数传递到函数内部，根据传入的参数创建对象，这一过程就产生了拷贝开销。再比如，在函数返回时，需要将函数内部创建的对象作为返回值返回，也需要进行对象拷贝，函数内的对象离开了作用域就会被销毁。

```c++
std::string foo() {
    std::string s = "hello world";
    return s;
}
```

##### 左值与右值

简单地说，C++ 中的左值（lvalue）是指向内存区域的变量，它可以出现在赋值表达式的左边或者右边；右值（rvalue）是不可寻址的常量，或者在表达式求值过程中无名的临时对象，只能出现在赋值表达式的右边。

##### 左值引用和右值引用

左值引用就是对变量的引用，对左值引用的操作就是对应内存的操作，一般声明语法是 `T&`。右值引用用于绑定一个临时的对象，并延长其生命周期到作用域结束，一般声明语法是 `T&&`。



### 移动构造和移动赋值

为了解决拷贝开销，C++11 引入了移动构造函数和移动赋运算符，它们的函数签名如下：

```c++
class MyClass {
public:
    MyClass(MyClass&& other);            // 移动构造
    MyClass& operator=(MyClass&& other); // 移动赋值
};
```

示例

```c++
#include <iostream>
#include <vector>

class Buffer {
    int* data;
    size_t size;
public:
    Buffer(size_t n) : size(n), data(new int[n]) {
        std::cout << "Construct\n";
    }
    ~Buffer() {
        delete[] data;
    }
    // 拷贝构造
    Buffer(const Buffer& other) : size(other.size), data(new int[other.size]) {
        std::cout << "Copy Construct\n";
        std::copy(other.data, other.data + size, data);
    }
    // 移动构造
    Buffer(Buffer&& other) noexcept : size(other.size), data(other.data) {
        std::cout << "Move Construct\n";
        other.data = nullptr;
        other.size = 0;
    }
};

int main() {
    std::vector<Buffer> vec;
    vec.reserve(2);

    vec.push_back(Buffer(10)); // 临时对象 => 移动构造
    Buffer b(20);
    vec.push_back(std::move(b)); // 显式转移 => 移动构造
}

```

运行结果

```bash
Construct
Move Construct
Construct
Move Construct
```

可以看到，上述两个操作都没有触发拷贝构造。第一次调用 `push_back()` 时传入的是临时构造的一个 `Buffer` 对象，它是右值，因此被 `Buffer(Buffer&& other)` 构造函数捕获，将这个临时对象的所有权转移到当前对象。

第二次调用 `push_back()` 时传入的是 `std::move(b)`，这里 `move` 是一个类型转换工具，用于将左值强制转换为右值引用，以便触发移动语义。因此，当前对象使用传入的 `b` 的右值引用进行移动构造，将 `b` 的所有权转移到当前对象。此时 `b` 的状态被置为资源已转移但仍可析构的状态，不应再使用原有的内容。

总而言之，C++ 移动语义通过右值引用支持资源转移，避免拷贝开销。`std::move()` 用于显式地出发移动语义，使用后的对象处于“已转移”的状态。C++ 的移动语义常用于函数参数传递、函数返回值传递、容器和字符串插入等场景。



