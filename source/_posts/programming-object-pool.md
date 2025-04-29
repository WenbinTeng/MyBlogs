---
title: programming-object-pool
date: 2025-04-16 15:48:33
tags:
---

# 对象池 Object Pool

如果需要在程序中频繁地创建和删除小对象，如果直接对内存进行操作可能会造成较大的时间开销和内部碎片，而对象池是针对这一场景的有效优化手段。通过提前分配一定数量的对象在对象池中，并在需要时从池中获取对象，使用完毕后再归还给池，可以减少动态分配内存的次数，避免频繁的创建和销毁操作，从而提高性能和资源利用率。



### 1. 对象池的设计思路

##### 1.1 基本功能

根据上述需求，分析实现一个对象池应该包含的基本功能如下：

- 构造函数中申请一批对象
- 析构函数中释放已申请的对象
- 提供获取和回收对象的接口
- 进行动态扩容

##### 1.2 高级功能

取决于实际应用场景，对象池可以实现以下高级功能：

- 并发安全：在多线程环境下，使用互斥锁保证并发访问安全。



### 2. 对象池的具体实现

##### 2.1 对象定义

以下是使用对象的定义，可以根据实际场景进行修改。

```c++
class Object {
private:
    // define variables
public:
    Object() {
        cout << "Object created." << endl;
    }
    ~Object() {
        cout << "Object destroyed." << endl;
    }
    void dosomething() {}
};
```

##### 2.2 对象块定义

假设内存池以 N 个对象为一组向内存申请空间，下面定义一个对象块进行管理。每个对象块创建时，会创建 N 个对象保存在数组里，并维护一个数组记录每个对象是否被分配。注意，在构造对象块时，使用 C++ 中的 operator new 向内存申请空间，每次分配时使用 placement new 对预分配内存的、空闲的对象进行分配。

```c++
template<typename T, size_t BlockSize>
class ObjectBlock {
private:
    void* _raw;
    std::vector<bool> _used;
    size_t _counter;
    
    T* get(size_t index) {
       	reinterpret_cast<T*>(static_cast<char*>(_raw) + index * sizeof(T));
    }
    
    size_t getIndex(T* ptr) {
        return (reinterpret_cast<char*>(ptr) - reinterpret_cast<char*>(_raw)) / sizeof(T); 
    }
    
public:
    ObjectBlock() {
        _raw = operator new(sizeof(T) * BlockSize);
        _used.resize(BlockSize, false);
        _counter = 0;
    }
    
    ~ObjectBlock() {
        for (size_t i = 0; i < BlockSize; i++) {
            if (_used[i]) {
                get(i)->~T();
            }
        }
        operator delete(_raw);
    }
    
    T* allocate() {
        for (size_t i = 0; i < BlockSize; i++) {
            if (!_used[i]) {
                _used[i] = true;
                _counter++;
                T* obj = new (get(i)) T();
                return obj;
            }
        }
        return nullptr;
    }
    
    int deallocate(T* ptr) {
        size_t index = getIndex(ptr);
        if (index >= BlockSize || !_used[index])
            return 1;
        ptr->~T();
        _used[index] = false;
        _counter--;
        return 0;
    }
    
    bool full() const { return _counter == BlockSize; }
    bool empty() const { return _counter == 0; }
};
```

##### 2.3 对象池定义

内存池使用数组管理申请的对象块。当需要向池申请一个对象时，遍历每个块，查询其是否已满：如果未满，则分配一个对象并返回指针；如果已满，则申请一块新的对象块添加到数组尾部，在新块中分配一个对象并返回指针。当需要向池释放一个对象时，并不会真正地释放这个对象的内存区域，而是在块中标记为未使用，池需要遍历每个块判断该对象是否属于当前块。

```c++
template<typename T, size_t BlockSize = 64>
class ObjectPool {
private:
    std::vector<std::unique_ptr<ObjectBlock<T, BlockSize>>> _blocks;
    std::mutex _mutex;
public:
    using Deleter = std::function<void(T*)>;
    using Ptr = std::unique_ptr<T, Deleter>;
    
    ObjectPool() = default;
    ~ObjectPool() = default;
    
    Ptr acquire() {
        T* rawPtr = nullptr;
        {
            std::lock_guard<std::mutex> lock(_mutex);
            for (auto& block : _blocks) {
                if (!block->full()) {
                    rawPtr = block->allocate();
                    break;
                }
            }
            if (!rawPtr) {
                auto newBlock = std::make_unique<ObjectBlock<T, BlockSize>>();
                rawPtr = newBlock->allocate();
                _blocks.push_back(std::move(newBlock));
            }
        }
        return Ptr(rawPtr, [this](T* p){ this->release(p); });
    }
    
    void release(T* obj) {
        std::lock_guard<std::mutex> lock(_mutex);
        for (auto& block : _blocks) {
            if (block->deallocate(obj) == 0)
                return;
        }
        assert(false && "Attempt to release object not in pool.");
    }
};
```

