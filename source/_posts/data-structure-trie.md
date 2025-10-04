---
title: data-structure-trie
date: 2025-10-04 17:17:32
tags:
---

# 字典树 Trie

字典树（Trie）是一种树形数据结构，主要用于高效地存储和检索字符串集合。它通过利用字符串的公共前缀来节省存储空间，常用于词典查询、自动补全等场景。



### 1. 什么是字典树

字典树的每条边代表一个字符，从根节点到某一节点的路径表示一个字符串。如果某节点被标记为结束节点，则表示该路径对应的字符串存在于集合中。



### 2. 字典树的基本操作

字典树的核心操作包括插入和查找，时间复杂度通常为字符串长度的线性级别，比暴力搜索更高效。

##### 2.1 树的构造

对于每个树的节点，应该包含以下字段：

- 指向子节点的数组 `children`，数组长度为所有字符的数量；
- 一个布尔变量 `end`，用于指定是否为字符串结尾。

##### 2.2 插入节点

```c++
void insert(std::string word) {
    Trie *node = this;
    for (char c : word) {
        c -= 'a';
        if (node->children[c] == nullptr) {
            node->children[c] = new Trie();
        }
        node = node->children[c];
    }
    node->end = true;
}
```

##### 2.3 查询

```c++
bool searchPrefix(std::string prefix) {
    Trie *node = this;
    for (char c : prefix) {
        c -= 'a';
        if (node->children[c] == nullptr) {
            return false;
        }
        node = node->children[c];
    }
    return node != nullptr && node->end;
}
```

##### 2.4 代码模版

```c++
class Trie {
public:
  	void insert(std::string word) {
        Trie *node = this;
        for (char c : word) {
            c -= 'a';
            if (node->children[c] == nullptr) {
                node->children[c] = new Trie();
            }
            node = node->children[c];
        }
        node->end = true;
    }

    bool searchPrefix(std::string prefix) {
        Trie *node = this;
        for (char c : prefix) {
            c -= 'a';
            if (node->children[c] == nullptr) {
                return false;
            }
            node = node->children[c];
        }
        return node != nullptr && node->end;
    }

private:
    std::vector<Trie*> children;
    bool end;
}
```



### 3. 示例

[208. 实现 Trie (前缀树)](https://leetcode.cn/problems/implement-trie-prefix-tree/)
