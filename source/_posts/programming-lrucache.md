---
title: programming-lrucache
date: 2025-09-02 17:06:36
tags:
---

# LRU Cache

当缓存容量固定时，我们需要一种淘汰策略：每次访问把数据标记为“最近使用”，当容量满了时淘汰“最久未使用”的那个，以提升缓存命中率。这就是 LRU 的核心。

以下是一个使用 C++ 实现的 LRU Cache 模版，拥有以下特性：

- 使用 `std::list` 维护一个缓存列表，头部的缓存块为最近使用的、尾部的缓存块为最久未使用的。每次读写都会将对应的缓存块提到头部，时间复杂度为 $O(1)$。
- 使用 `std::unordered_map` 维护一个查找表，根据关键字快速查找缓存块在列表中的位置，时间复杂度为 $O(1)$。

```c++
#include <list>
#include <unordered_map>

template<typename K, typename V>
class LRUCache {
public:
    LRUCache(size_t capacity) : _cap(capacity) {
        if (_cap <= 0)
            throw "ERROR: capacity must be > 0";
    }
    ~LRUCache() = default;

    std::unique_ptr<V> get(const K& key) {
        if (!_lookup.count(key)) {
            return nullptr;
        }
        auto it = _lookup[key];
        update(it);
        return std::make_unique<V>(it->second);
    }

    void put(const K& key, const V& value) {
        if (_lookup.count(key)) {
            auto it = _lookup[key];
            it->second = value;
            update(it);
            return;
        }
        if (_lru.size() == _cap) {
            int k = _lru.back().first;
            _lookup.erase(k);
            _lru.pop_back();
        }
        _lru.push_front({key, value});
        _lookup[key] = _lru.begin();
    }

private:
    using node_t = std::pair<K, V>;
    using iter_t = typename std::list<node_t>::iterator;

    size_t _cap;
    std::list<node_t> _lru;
    std::unordered_map<K, iter_t> _lookup;

    void update(iter_t it) {
        _lru.splice(_lru.begin(), _lru, it);
    }
};
```

