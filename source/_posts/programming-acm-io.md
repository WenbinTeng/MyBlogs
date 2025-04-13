---
title: programming-acm-io
date: 2025-04-13 08:57:32
tags:
---

# ACM 模式的输入输出方法

在各大公司的机试中，常见的答题模式有 **核心模式** 与 **ACM 模式**。核心模式通常是一些在线测试平台（如力扣、牛客等）上使用的答题模式，需要答题者在函数体完成代码，函数的输入输出都是固定的，以供测试代码调用。ACM 模式需要答题者自行处理输入输出，通常以标准输入输出格式表示。下面以 C++ 为例展示 ACM 模式处理输入输出的模板。

##### 1. 固定长度一维数组

输入格式：

```
4
1 2 3 4
```

处理模板：

```c++
#include <iostream>
#include <vector>
using namespace std;
int main() {
	int n;
    cin >> n;
    vector<int> arr(n);
    for (int i = 0; i < n; i++) {
        int a;
        cin >> a;
        arr[i] = a;
    }
    for (int i = 0; i < n; i++) {
        cout << arr[i] << " ";
	}
    cout << endl;
    return 0;
}
```

##### 2. 不定长度一维数组

输入格式：

```
1 2 3 4
```

处理模板：

```c++
#include <iostream>
#include <vector>
using namespace std;
int main() {
    int a;
    vector<int> arr;
    while (cin >> a) {
        arr.push_back(a);
       	if (cin.get() == '\n')
            break;
    }
    for (int i = 0; i < arr.size(); i++) {
        cout << arr[i] << " ";
	}
    cout << endl;
    return 0;
}
```

##### 3. 二维数组

输入格式：

```
2 4
1 2 3 4
5 6 7 8
```

处理模板：

```c++
#include <iostream>
#include <vector>
using namespace std;
int main() {
    int m, n;
    cin >> m >> n;
    vector<vector<int>> arr(m, vector<int>(n));
    for (int i = 0; i < m; i++) {
        for (int j = 0; j < n; j++) {
        	int a;
            cin >> a;
            arr[i][j] = a;
        }
    }
    for (int i = 0; i < m; i++) {
        for (int j = 0; j < n; j++) {
            cout << arr[i][j] << " ";
        }
        cout << endl;
    }
    return 0;
}
```

##### 4. 不定数目多个字符串

输入格式：

```
a b c ab bc abc
```

处理模板：

```c++
#include <iostream>
#include <vector>
#include <string>
using namespace std;
int main() {
    string str;
    vector<string> arr;
    while (cin >> str) {
        arr.push_back(str);
        if (cin.get() == '\n')
            break;
    }
    for (int i = 0; i < arr.size(); i++) {
        cout << arr[i] << endl;
	}
    return 0;
}
```

