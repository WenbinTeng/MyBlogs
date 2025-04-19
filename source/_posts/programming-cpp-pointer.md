---
title: programming-cpp-pointer
date: 2025-04-19 22:23:38
tags:
---

# C++ 智能指针与引用 C++ Smart Pointer and Reference

在 C 语言编程中，使用裸指针（raw pointer）几乎是处理动态内存的唯一方式。然而，随着 C++ 的不断演进，特别是在 C++11 及之后的标准中，智能指针（`std::unique_ptr`, `std::shared_ptr`, `std::weak_ptr`）和引用（`&`）成为了更安全、更现代的替代品。



### 使用 C 风格指针的风险

C 风格的指针虽然灵活，但也伴随着许多问题：

- **内存泄漏**：每一个 `new` 必须对应一个 `delete`，不小心忘记释放就可能导致内存泄漏。
- **空指针**：在释放指针后继续访问，会引发未定义行为。
- **野指针：未初始化的指针可能指向随机内存。



### 使用 C++ 智能指针

智能指针是一种模板类，封装了动态分配对象的指针，并在其生命周期结束时自动释放资源。

1. **内存自动释放**
    `std::unique_ptr` 在析构时自动调用 `delete`；`std::shared_ptr` 通过引用计数管理生命周期。

2. **异常安全**
    在栈上管理智能指针，无论函数怎样退出，析构都会正确执行，不用手动释放。

3. **所有权语义清晰**

   - `std::unique_ptr<T>`：独占所有权，不可复制，仅可移动。

     ```c++
     void foo() {
         std::unique_ptr<int> up(new int(100));
         *up += 1;
         // auto up2 = up; 		  	// Error: Not duplicable.
         auto up2 = std::move(up); 	// OK: Pass ownership.
     }
     ```

   - `std::shared_ptr<T>`：共享所有权，内部引用计数；当最后一个 `shared_ptr` 销毁时释放资源。

     ```c++
     void foo() {
         std::shared_ptr<int> sp1 = std::make_shared<int>(200);
         {
             std::shared_ptr<int> sp2 = sp1; // Reference count +1.
             std::cout << *sp2 << "\n";
         } // sp2 leaves the scope, reference count -1.
         // When the last sp leaves the scope, the delete operator is called automatically.
     }
     ```

   - `std::weak_ptr<T>`：配合 `shared_ptr` 使用，不影响引用计数，可安全检测对象是否已销毁。

     ```c++
     void foo() {
         std::shared_ptr<int> sp(new int(10));
         std::weak_ptr<int> wp(sp);
         std::cout << wp.use_count() << std::endl; // wp does not increase reference count.
         std::cout << *(wp.lock()) << std::endl; // Access the object pointed by sp
     }
     ```

     

### 函数传参时应传递引用以代替传递指针

在 C++ 编程中，**如果函数中仅仅想使用对象内的资源，那么应该向函数传递引用而不是传递智能指针，因为函数内不需要智能指针声明周期的相关语义。**

```c++
void process(int& value) {
    // value will always be ready.
    value += 10;
}
int main() {
    int x = 5;
    process(x);      		// OK
    // process(nullptr);  	// Error: null pointer.
}
```



### 易混淆的点：智能指针与引用结合

参考 [C++ Core Guidelines](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#S-resource)：

| Function signature                     | Semantic                            | Rule |
| -------------------------------------- | ----------------------------------- | ---- |
| `func(std::unique_ptr<Widget>)`        | func takes ownership                | R.32 |
| `func(std::unique_ptr<Widget>&)`       | func meant to reseat Widget         | R.33 |
| `func(std::shared_ptr<Widget>)`        | func shares ownership               | R.34 |
| `func(std::shared_ptr<Widget>&)`       | func might reseat Widget            | R.35 |
| `func(const std::shared_ptr<Widget>&)` | func might retain a reference count | R.36 |

