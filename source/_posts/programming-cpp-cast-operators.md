---
title: programming-cpp-cast-operators
date: 2025-04-19 23:33:17
tags:
---

# C++ 类型转换 C++ Type Conversion

在 C 语言编程中，类型转换的方式非常简单，只有两种方法。第一种是隐式类型转换，这由编译器自动进行，比如：

```c
float x = 10; // Convert int to float
```

第二种是显式类型转换，由程序指定，比如：

```c
int x = (int)3.14; // Convert float to int
```

虽然这些类型转换比较方便，但是缺少类型安全机制，指针转换过程中容易出现 bug 而编译器无法给出相应的警告。



### C++ 强类型转换

C++ 中引入了四种类型转换操作符以应对不同的场景：

- `static_cast<T>(expression)`：用于相关类型之间的转换，如类型之间、继承类指针之间的转换。

  ```c++
  double d = 3.14;
  int i = static_cast<int>(d);
  ```

- `const_cast<T>(expression)`：用来添加或移除 `const`、`volatile` 限定符，常用于需要去除 `const` 的 API 接口场景。

  ```c++
  const int* p = new int(10);
  int* q = const_cast<int*>(p);
  ```

- `reinterpret_cast<T>(expression)`：用于底层类型转换，如将 `int*` 转成 `float*`，或者整数与指针之间的转换。该类型转换编译器不会报错，需要慎用。

  ```c++
  int* p = new int(5);
  char* c = reinterpret_cast<char*>(p);
  ```

- `dynamic_cast<T>(expression)`：只用于有虚函数的多态类之间的指针或引用类型转换，可以做运行时类型检查（RTTI）。转换失败时，返回 `nullptr` 或抛出异常。

  ```c++
  class Base { virtual void foo() {} };
  class Derived : public Base {};
  Base* b = new Derived;
  Derived* d = dynamic_cast<Derived*>(b);
  ```
