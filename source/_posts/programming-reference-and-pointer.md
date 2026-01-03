---
title: programming-reference-and-pointer
date: 2025-10-08 08:32:28
tags:
---

# C++ 引用和指针的区别 C++ Reference and Pointer

在 C++ 中，指针（Pointer）和引用（Reference）都用于间接访问变量，但是它们在语法和语义上都有显著的差异。指针是一个变量，它可以保存另一个变量的内存地址；引用是另一个变量的别名，由编译器进行处理，使用时与使用原变量无异。

指针和引用使用的一些差异如下：

**初始化**

指针：可以先定义后赋值，也可以指向不同的对象。

```c++
int a = 10, b = 20;
int* p = &a;
p = &b; // 改变指向
```

引用：必须在定义时就进行初始化，绑定到一个特定的对象，并且不能绑定到其他的对象。

```c++
int a = 10, b = 20;
int& r = a;  
r = b; // 修改的是 a 的值，而不是重新绑定，等价于 a=b
```

**可空性**

指针：可以被定义为 `nullptr`，表示不指向任何对象。

引用：必须绑定有效对象，不能为 `null`，否则会产生为定义的行为。

##### 内存占用

指针：由于存储的是内存地址，因此需要占用内存空间，一般为 32 bit 或 64 bit。

引用：语义上不占用内存空间，实际可能由编译器实现为一个隐藏指针。

这里我们参考一篇博客： [C++ 中指针和引用的区别（汇编分析） | 编程指北-计算机学习指南](https://csguide.cn/cpp/memory/difference_of_pointers_and_ref.html) 。

参考以下使用引用和指针实现的 `swap` 函数。

```c++
void swap(int &a, int &b) {
    int temp = a;
    a = b;
    b = temp;
}

void swap(int *a, int *b) {
    int temp = *a;
    *a = *b;
    *b = temp;
}
```

引用版本 `swap()` 函数的汇编代码如下：

```assembly
# void swap(int &a, int &b)
__Z4swapRiS_:                           ## @_Z4swapRiS_
        .cfi_startproc
## %bb.0:
        pushq   %rbp
        .cfi_def_cfa_offset 16
        .cfi_offset %rbp, -16
        movq    %rsp, %rbp          
        .cfi_def_cfa_register %rbp
        movq    %rdi, -8(%rbp)         # 传入的第一个参数存放到%rbp-8  （应该是采用的寄存器传参，而不是常见的压栈）
        movq    %rsi, -16(%rbp)        # 第二个参数 存放到 %rbp-16
        movq    -8(%rbp), %rsi         # 第一个参数赋给 rsi
        movl    (%rsi), %eax           # 以第一个参数为地址取出值赋给eax，取出*a暂存寄存器
        movl    %eax, -20(%rbp)        # temp = a
        movq    -16(%rbp), %rsi        # 将第二个参数重复上面的
        movl    (%rsi), %eax
        movq    -8(%rbp), %rsi    
        movl    %eax, (%rsi)           # a = b
        movl    -20(%rbp), %eax        # eax = temp
        movq    -16(%rbp), %rsi
        movl    %eax, (%rsi)           # b = temp
        popq    %rbp
        retq
        .cfi_endproc
                                        ## -- End function
```

指针版本 `swap()` 函数的汇编代码如下：

```assembly
# void swap(int *a, int *b)
__Z4swapPiS_:                           ## @_Z4swapPiS_
        .cfi_startproc
## %bb.0:
        pushq   %rbp
        .cfi_def_cfa_offset 16
        .cfi_offset %rbp, -16
        movq    %rsp, %rbp
        .cfi_def_cfa_register %rbp
        movq    %rdi, -8(%rbp)
        movq    %rsi, -16(%rbp)
        movq    -8(%rbp), %rsi
        movl    (%rsi), %eax
        movl    %eax, -20(%rbp)
        movq    -16(%rbp), %rsi
        movl    (%rsi), %eax
        movq    -8(%rbp), %rsi
        movl    %eax, (%rsi)
        movl    -20(%rbp), %eax
        movq    -16(%rbp), %rsi
        movl    %eax, (%rsi)
        popq    %rbp
        retq
        .cfi_endproc
                                        ## -- End function
```

可以看到，两个函数的汇编代码几乎完全一致。引用所赋的初值也就是绑定对象的内存地址，访问和修改对象内容也是通过这个内存地址完成的。

既然引用和指针的底层实现机制几乎没有区别，那么为什么我们需要使用引用呢？我认为有以下两点：

- 便捷性：能够让使用者使用引用与使用对象本身一致，由编译器自动进行取地址、解引用的操作。
- 安全性：引用必须初始化，且不能更换绑定，避免了悬空指针和野指针的问题，这在接口设计中更可靠，要求必须传入合法的对象。
