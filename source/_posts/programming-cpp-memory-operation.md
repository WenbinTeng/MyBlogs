---
title: programming-cpp-memory-operation
date: 2025-04-18 22:53:42
tags:
---

# C++ 中的 new 和 delete 操作 The new and delete Operations in C++

在 C++ 中，`new` 是我们最常用的内存分配方式之一。很多初学者会以为 `new` 只是一个语法糖，用来代替 C 语言的 `malloc`，但实际上，`new` 背后涉及的是一整套机制，其中包括 `operator new`、`placement new` 等概念。



### 1. new

我们一般使用以下语句创建一个对象：

```c++
MyClass* obj = new MyClass();
```

这一段代码实际上完成了三个操作：

1. **分配内存**：调用标准库函数 `operator new` 来分配相应的内存空间，这一默认函数可以重载。
2. **构造对象**：在分配的内存空间上调用 `placement new` 构建对象，这会调用对象相应的构造函数。
3. **返回指针**：返回分配并构造好的对象指针。

可以理解为以下两步代码：

```c++
void* mem = operator new(sizeof(MyClass));
MyClass* obj = new (mem) MyClass();
```



### 2. operator new

`operator new` 是一个可以重载的函数，负责在堆上分配内存。其默认定义如下：

```c++
void* operator new(size_t size)
```

函数返回的是 `void*` 类型的指针，这个指针指向的是一块未初始化的内存区域。`operator new` 函数不知道类的构造函数，它只负责内存空间的申请，并返回对应的指针。




### 3. placement new

`placement new` 能直接调用构造函数以在已分配的内存空间上初始化一个对象。其语法如下：

```c++
new (address) Type(args...);
```

注意，`placement new` 分配的对象必须手动调用析构函数，而不能使用 `delete` 操作符来回收这块内存，因为 `delete` 会释放内存空间，而这块内存是你自己管理的。



### 4. deletion and deallocation

使用 `operator new` 和 `placement new` 分配的对象应该使用以下步骤删除对象并释放内存：

```c++
void* raw = operator new(sizeof(MyClass));
MyClass* p = new (raw) MyClass();
p->~MyClass();
operator delete raw;
```

