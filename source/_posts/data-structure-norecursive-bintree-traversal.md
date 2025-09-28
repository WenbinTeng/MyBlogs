---
title: programming-norecursive-bintree-traversal
date: 2025-09-28 10:37:24
tags:
---

# 二叉树的非递归方式遍历

在实现二叉树遍历时，使用递归的方式是比较直观的方式。然而递归依赖系统调用栈，一方面在树深度很大时可能导致栈溢出，另一方面在工程实践中往往需要更灵活的控制方式。于是，**非递归遍历**成为一种必要补充：通过显式地维护栈或队列来模拟递归过程，从而在不依赖系统栈的情况下完成遍历。本博客将介绍一些非递归的方式实现二叉树遍历的方法。



### 1. 节点定义

```c++
struct TreeNode {
    int val;
    TreeNode *left;
    TreeNode *right;
    TreeNode() : val(0), left(nullptr), right(nullptr) {}
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
    TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
};
```



### 2. 先序遍历

二叉树的先序遍历遵循"根-左-右"的顺序。使用非递归方式实现时，我们需要借助**栈**来模拟递归过程。具体思路如为：

1. 从根节点开始，将节点压入栈中；
2. 执行以下步骤直至栈为空：
   - 取出栈顶节点并访问；
   - 如果该节点有右节点，则将右节点压栈；
   - 如果该节点有左节点，则将左节点压栈。

```c++
void preorder(TreeNode *root) {
    if (root == nullptr)
        return;

    stack<TreeNode *> st;
    st.push(root);

    while (!st.empty()) {
        TreeNode *node = st.top();
        st.pop();

        cout << node->val << endl;

        if (node->right != nullptr) {
            st.push(node->right);
        }
        if (node->left != nullptr) {
            st.push(node->left);
        }
    }
}
```



### 3. 中序遍历

二叉树的中序遍历遵循"左-根-右"的顺序。使用非递归方式实现时，同样需要借助**栈**来模拟递归过程。具体思路如下：

1. 从根节点开始，将所有左节点依次压入栈中；
2. 当没有左子节点时，弹出栈并访问；
3. 转向该节点的右子树，重复步骤 1-2；
4. 直到栈为空且当前节点为 null 时结束。

```c++
void inorderTraversal(TreeNode *root) {
    stack<TreeNode *> st;
    TreeNode *curr = root;

    while (curr != nullptr || !st.empty()) {
        while (curr != nullptr) {
            st.push(curr);
            curr = curr->left;
        }

        curr = st.top();
        st.pop();
        cout << curr->val << " ";

        curr = curr->right;
    }
}
```



### 4. 后续遍历

二叉树的后序遍历遵循"左-右-根"的顺序，是非递归遍历中最复杂的一种，因为需要确保在访问根节点之前，左右子树都已被访问。

##### 4.1 方法一：使用两个栈

使用两个栈实现后序遍历的具体思路为：

1. 初始化两个栈 `s1` 和 `s2`，并将根节点压入 `s1`；
2. 循环执行直到 `s1` 为空：
   - 将 `s1` 栈顶元素弹出，并压入 `s2`；
   - 将弹出节点的左子节点和右子节点压入 `s1` 
3. 依次弹出 `s2` 中的所有节点并访问。

```c++
void postorderTraversal(TreeNode *root) {
    if (root == nullptr)
        return;

    stack<TreeNode *> s1, s2;
    s1.push(root);

    while (!s1.empty()) {
        TreeNode *node = s1.top();
        s1.pop();
        s2.push(node);

        if (node->left != nullptr) {
            s1.push(node->left);
        }
        if (node->right != nullptr) {
            s1.push(node->right);
        }
    }

    while (!s2.empty()) {
        cout << s2.top()->val << endl;
        s2.pop();
    }
}
```

##### 4.2 方法二：使用一个栈

使用一个栈实现后序遍历的具体思路为：

1. 从根节点开始，将所有左子节点压入栈中；
2. 使用一个指针记录前一个访问的节点；
3. 循环检查栈顶节点：
   - 如果当前的节点未访问，则转入右子树；
   - 如果当前的节点已访问，则访问当前节点，并将指针指向当前节点。

```c++
void postorderTraversal2(TreeNode *root) {
    if (root == nullptr)
        return;

    stack<TreeNode *> st;
    TreeNode *prev = nullptr;
    TreeNode *curr = root;

    while (curr != nullptr || !st.empty()) {
        while (curr != nullptr) {
            st.push(curr);
            curr = curr->left;
        }

        TreeNode *topNode = st.top();

        if (topNode->right != nullptr && topNode->right != prev) {
            curr = topNode->right;
        } else {
            cout << topNode->val << endl;
            prev = topNode;
            st.pop();
        }
    }
}
```

