---
title: programming-cpp-mutex
date: 2025-04-24 09:03:48
tags:
---

# C++ 互斥锁 C++ Mutex

在多线程编程中，使用互斥锁能够避免数据竞争。在 C++ 标准库中有 `std::mutex` 的互斥锁实现来帮助我们处理线程同步。



### 1. 互斥锁的基本用法

在 C++ 11 中，引入了 RAII （Resource Acquisition Is Initialization）风格的锁管理类 `std::lock_guard`，其使用方法为：

```c++
#include <iostream>
#include <vector>
#include <thread>
#include <mutex>

std::mutex mtx;
int counter = 0;

void increment_old() {
    mtx.lock();
    counter++;
    mtx.unlock();
}

void increment() {
    // Classic lock and unlock mod.
    // mtx.lock();
    // counter++;
    // mtx.unlock();
    
    // Auto locks within the scope and unlocks when out of the scope.
    std;:lock_guard<std::mutex> lg(mtx);
    counter++;
}

int main() {
    std::vector<std::thread> threads;
    for (int i = 0; i < 3; i++)
        threads.emplace_back(increment);
    for (auto& t : threads)
        t.join();
    std::cout << counter << std::endl;
    return 0;
}
```

相比于传统的的手动加锁 `lock()` 和手动解锁 `unlock()` 方式，`lock_guard` 可以避免在临界区发生异常时忘记解锁而造成的死锁或者锁泄露。



### 2. 使用 "unique_lock" 进行更细粒度的控制

`lock_guard` 会在构造函数内进行加锁，在析构函数内进行解锁。然而，如果其作用域过大，可能会影响程序的效率。此时，我们可以使用 `unique_lock` 对加锁过程进行更细粒度的控制。

`unique_lock` 支持延迟加锁、尝试加锁、定时尝试加锁、递归锁定、所有权转移等操作，其基本使用方法如下：

- 默认构造：对传入的 `mutex` 尝试上锁。

  ```c++
  std::mutex mtx;
  std::unique_lock<std::mutex> ul(mtx);
  ```

- 延迟加锁：传递参数 `std::defer_lock` 来指定创建 `unique_lock` 对象时不立即上锁。

  ```c++
  {
      std::mutex mtx;
      std::unique_lock<std::mutex> ul(mtx, std::defer_lock);
      // mtx is currently unlocked.
      do_something();
      ul.lock();
      shared_data++;
      // auto unlock when leaving scope.
  }
  ```

- 尝试加锁：传递参数 `std::try_to_lock` 来对传入的 `mutex`  尝试加锁，即使未能锁定也不会阻塞当前线程。或者，可以先延迟加锁，然后在需要的时候使用 `try_lock()` 方法来尝试加锁。

  ```c++
  {
      std::mutex mtx;
      // std::unique_lock<std::mutex> ul(mtx, std::try_to_lock);
      std::unique_lock<std::mutex> ul(mtx, std::defer_lock);
      // lock mtx if available.
      if (ul.try_lock()) {
          // acquired.
          shared_data++;
          // auto unlock when leaving scope.
      } else {
          // failed to acquire the lock.
      }
  }
  ```



### 3. 使用 "shared_lock" 提升并发性能

对于只读场景，可以使用读写锁 `shared_lock` 来共享临界区资源，提升并发访问性能。一个经典的读写场景如下：

```c++
#include <iostream>
#include <vector>
#include <thread>
#include <shared_mutex>

std::shared_mutex rw_mutex;
int data = 0;

void reader() {
    std::shared_lock<std::shared_mutex> sl(rw_mutex);
    std::cout << "reading data = " << data << std::endl;
}

void writer() {
    std::unique_lock<std::shared_mutex> ul(rw_mutex);
    data++;
    std::cout << "writing data = " << data << std::endl;
}

int main() {
    std::vector<std::thread> threads;
    for (int i = 0; i < 3; i++)
        threads.emplace_back(reader);
    threads.emplace_back(writer);
    for (auto &t : threads)
        t.join();
    return 0;
}
```

