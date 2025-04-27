---
title: programming-cpp-thread
date: 2025-04-27 09:44:39
tags:
---

# C++ 线程 C++ Thread

在 C++ 编程中一般使用 `std::thread` 来进行创建多个线程，实现并发执行、提升程序性能。



### 1. 线程的创建和启动

最简单的方式就是将一个可调用对象（函数、函数对象、lambda 表达式）传给 `std::thread` 构造函数：

```c++
#include <iostream>
#include <thread>

void hello() {
    std::cout << "Hello from thread!" << std::endl;
}

int main() {
    std::thread t(hello);
    t.join();
    return 0;
```

上述过程有几个关键点：

- 线程在构造完毕后交由操作系统进行调度，常用的构造函数有：
  - 默认构造函数： `thread() noexcept;`
  - 初始化构造函数：`explicit thread(FN&& fn, Args&&... args);`
  - 拷贝构造函数：`thread(const thread&) = delete;`
  - 移动构造函数：`thread(thread&& x) noexcept;`
- 可被 `joinable` 的线程对象必须在它们被销毁之前被主线程 `join` （阻塞等待）或者将其设置为 `deteched`（分离，不再管理）。否则，`std::terminate()` 会被触发，程序异常终止。



### 2. 重要的成员函数

- `get_id()`：获取线程的 ID，返回一个 `std::thread::id` 对象。

- `joinable()`：检查当前线程是否被 `join`。

- `detach()`：将当前线程对象所代表的执行实例与此线程对象分离，使得线程可以单独执行。

- `swap()`：交换两个线程对象所代表的底层句柄，参数是两个线程对象；
- `std::this_thread::getid()`：获取当前线程的ID；
- `std::this_thread::yield()`：当前线程放弃执行，操作系统调度另一线程继续执行；
- `sleep_until()`：线程休眠至某个指定的时刻，才被重新唤醒；
- `sleep_for()`： 线程休眠某个指定的时间片，才被重新唤醒；



### 3. 使用线程的注意事项

1. **缩小并发边界**：仅并发真正耗时、可并行的部分；过度并行会增加上下文切换开销。

2. **避免过度细粒度线程**：启动/销毁线程代价大，频繁创建数量众多的小任务不划算。

3. **保持数据结构线程安全**：共享可变数据必须受同步原语保护，或设计为无锁/不可变对象。

4. **统一异常处理**：在线程入口捕获所有异常，避免 `std::terminate`。

   ```c++
   std::thread t([]{
       try {
           do_work();
       } catch (...) {
           log_error();
       }
   });
   ```

5. **合理利用硬件资源**：可调用 `std::thread::hardware_concurrency()` 获取推荐并发数，根据负载动态调整线程数。
