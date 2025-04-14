---
title: algorithm-kmp
date: 2025-04-14 09:20:29
tags:
---

# KMP算法 KMP Algorithm

子串的定位操作通常称为串的模式匹配，它求的是子串（常称为模式串）在主串中的位置。暴力匹配算法会产生 $O(nm)$ 的时间复杂度，其中 $n$ 是主串长度，$m$ 是模式串长度。而 KMP 算法能将匹配的时间复杂度降低到线性时间 $O(n+m)$，本文将介绍 KMP 算法的核心思想与代码示例。

### 1. KMP 算法思想

KMP 算法的关键是避免重复的比较。它通过一个预处理的步骤，构建一个**部分匹配表**（Partial Match Table），使用这个表来指定每次匹配后模式串的移动距离，以跳过一些主串中显然不匹配的位置。

##### 1.1 字符串的前缀、后缀与部分匹配值

- 前缀指除最后一个字符以外，字符串的所有头部子串；
- 后缀指除最前一个字符以外，字符串的所有尾部子串；
- 部分匹配值是每个位置前的最长相等前后缀长度。

以模式串 `abcac` 为例，它的部分匹配表为：

| Index   | 1    | 2    | 3       | 4            | 5                  |
| ------- | ---- | ---- | ------- | ------------ | ------------------ |
| Pattern | a    | b    | c       | a            | c                  |
| Prefix  | {}   | {a}  | {a, ab} | {a, ab, abc} | {a, ab, abc, abca} |
| Suffix  | {}   | {b}  | {c, bc} | {a, ca, bca} | {c, ac, cac, bcac} |
| PM      | 0    | 0    | 0       | 1            | 0                  |

##### 1.2 匹配过程

设主串为 `ababcabcacbab`，模式串为 `abcac`，以下是字符串的匹配过程。设 $i$ 为主串匹配位置的下标，$j$ 为模式串匹配位置的下标，则移动位数 $d$ 的计算公式为：
$$
d = (j-1) - PM(j-1)
$$
第一趟比较过程：

| s    | a    | b    | a    | b    | c    | a    | b    | c    | a    | c    | b    | a    | b    |
| ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| p    | a    | b    | c    |      |      |      |      |      |      |      |      |      |      |

比较发现 $j = 2$ 位置的字符 a 与 c 不匹配，查表可知最后一个匹配字符 b 的部分匹配值为 0，则移动位数为 $d = 2 - 0 = 2$，将模式串向后移动两位。

第二趟比较过程：

| s    | a    | b    | a    | b    | c    | a    | b    | c    | a    | c    | b    | a    | b    |
| ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| p    |      |      | a    | b    | c    | a    | c    |      |      |      |      |      |      |

比较发现 $j=4$ 位置的字符 b 与 c 不匹配，计算移动位置 $d = 4 - 1 = 3$。

第三趟比较过程：

| s    | a    | b    | a    | b    | c    | a    | b    | c    | a    | c    | b    | a    | b    |
| ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| p    |      |      |      |      |      | a    | b    | c    | a    | c    |      |      |      |

模式串全部比较完成，匹配成功。



### 2. KMP 算法实现

##### 2.1 构建部分匹配表

```c++
std::vector<int> buildPMT(std::string& pattern) {
    int m = pattern.size();
    std::vector<int> pmt(m, 0);
    
    int len = 0;
    for (int i = 1; i < m; i++) {
       	while (len > 0 && pattern[i] != pattern[len]) {
            len = pmt[len - 1];
        }
        if (pattern[i] == pattern[len])
            len++;
        pmt[i] = len;
    }
    
    return pmt;
}
```

##### 2.2 比较

```c++
std::vector<int> KMPSearch(std::string& text, std::string& pattern) {
    int n = text.size();
    int m = pattern.size();
    std::vector<int> pmt = buildPMT(pattern);
    std::vector<int> match;
    
    int i = 0;
    int j = 0;
    
    while (i < n) {
        if (text[i] == pattern[j]) {
            i++;
            j++;
        }
        if (j == m) {
            match.push_back(i - j);
            j = pmt[j - 1];
        } else if (i < n && text[i] != pattern[j]) {
            if (j > 0)
                j = pmt[j - 1];
            else
                i++;
        }
    }
    
    return match;
}
```

