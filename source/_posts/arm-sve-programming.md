---
title: arm-sve-programming
date: 2025-08-22 19:30:59
tags:
---

# ARM SVE 编程

编写或生成 SVE 代码由几种方法：

- 编写 SVE 汇编代码；
- 使用 SVE intrinsics 进行编程；
- 自动向量化；
- 使用 SVE 优化的库。

下面我们将详细介绍这四种方法。

### 1. 编写 SVE 汇编代码

我们可以在 C/C++ 代码中以内联汇编形式编写 SVE 指令，或在汇编源文件中编写完整函数。例如：

```c
        .globl  subtract_arrays         // -- Begin function 
        .p2align        2 
        .type   subtract_arrays,@function 
subtract_arrays:               // @subtract_arrays 
        .cfi_startproc 
// %bb.0: 
        orr     w9, wzr, #0x400 
        mov     x8, xzr 
        whilelo p0.s, xzr, x9 
.LBB0_1:                       // =>This Inner Loop Header: Depth=1 
        ld1w    { z0.s }, p0/z, [x1, x8, lsl #2] 
        ld1w    { z1.s }, p0/z, [x2, x8, lsl #2] 
        sub     z0.s, z0.s, z1.s 
        st1w    { z0.s }, p0, [x0, x8, lsl #2] 
        incw    x8 
        whilelo p0.s, x8, x9 
        b.mi    .LBB0_1 
// %bb.2: 
        ret 
.Lfunc_end0: 
        .size   subtract_arrays, .Lfunc_end0-subtract_arrays 
        .cfi_endproc T
```

若混合使用高级语言和汇编编写的函数，必须熟悉针对 SVE 更新的 ABI 标准。《Arm 架构过程调用标准》（AAPCS）规定了数据类型和寄存器分配规则，要求：

- `Z0-Z7` 和 `P0-P3` 用于传递可扩展向量参数和结果；
- `Z8-Z15` 和 `P4-P15` 由被调用方保存；
- 所有其他向量寄存器（ `Z16-Z31` ）均可被被调用函数修改，调用函数需在必要时负责备份和恢复这些寄存器。

### 2. 使用 SVE instrinsics 进行编程

SVE instrinsics 是编译器支持的内联函数，可被替换为对应的指令。程序员可直接在 C 和 C++等高级语言中调用这些指令函数。针对 SVE 的 ACLE（Arm C 语言扩展）定义了可用的 SVE 指令函数、其参数及功能。支持 ACLE 的编译器在编译过程中会将 instrinsics 替换为关联的 SVE 指令。使用 ACLE instrinsics 时，必须包含头文件 "arm_sve.h"，该文件包含可在 C/C++ 中使用的向量类型和指令函数（针对 SVE）列表。每种数据类型描述了向量中元素的大小和数据类型：

- `svint8_t svuint8_t`
- `svint16_t svuint16_t svfloat16_t`
- `svint32_t svuint32_t svfloat32_t`
- `svint64_t svuint64_t svfloat64_t`

例如， `svint64_t` 表示 64 位有符号整数的向量， `svfloat16_t` 表示半精度浮点数的向量。

以下 C 代码示例是经过手动优化的 SVE 内联函数：

```c
//intrinsic_example.c
#include <arm_sve.h>
svuint64_t uaddlb_array(svuint32_t Zs1, svuint32_t Zs2)
{
    // widening add of even elements
    svuint64_t result = svaddlb(Zs1, Zs2);
    return result;
}
```

包含 arm_sve.h 的源代码可以像使用数据类型进行变量声明和函数参数一样使用 SVE 向量类型。使用 Arm C/C++编译器编译代码并以支持 SVE 的 Armv8-A 架构为目标时，请使用：

```bash
armclang -O3 -S -march=armv8-a+sve -o intrinsic_example.s intrinsic_example.c
```

上述命令会生成以下汇编代码：

```
//instrinsic_example.s
uaddlb_array:                           // @uaddlb_array
        .cfi_startproc
// %bb.0:
        uaddlb  z0.d, z0.s, z1.s
        ret
```

### 3. 自动向量化

Arm Compiler for Linux 以及 Arm 平台的 GNU 编译器支持使用 SVE 指令对 C、C++和 Fortran 循环进行向量化。要生成 SVE 代码，需选择适当的编译器选项。例如，当 `armclang` 使用 `-march=armv8-a+sve` 选项时， `armclang` 也会默认采用 `-fvectorize` 和 `-O2` 选项。如需启用支持 SVE 的库版本，请将 `-march=armv8-a+sve` 与 `-armpl=sve` 组合使用。关于编译器优化选项的更多信息，请参阅编译器开发者与参考指南，或编译器手册页。

### 4. 使用 SVE 优化的库

可以使用针对 SVE 高度优化的库进行计算加速，例如 Arm 性能库（[Arm Performance Libraries](https://developer.arm.com/tools-and-software/server-and-hpc/compile/arm-compiler-for-linux/arm-performance-libraries)）和 Arm 计算库（[Arm Compute Library](https://developer.arm.com/ip-products/processors/machine-learning/compute-library)）。Arm 性能库包含针对 BLAS、LAPACK、FFT、稀疏线性代数以及 libamath 优化的数学函数的高度优化实现。要能够链接任何 Arm 性能库函数，您必须安装 Arm Allinea Studio 并在代码中包含 armpl.h。若使用 Arm Compiler for Linux 和 Arm 性能库构建应用程序，必须在命令行中指定 `-armpl=<arg>` 。如果使用 GNU 工具，则需在链接器命令行中包含 Arm 性能库的安装路径，使用 `-L<armpl_install_dir>/lib` ，并指定与 Arm Compiler for Linux 的 `armpl=<arg>` 选项等效的 GNU 选项，即 `-larmpl_lp64` 。更多信息，请参考 Arm 性能库入门指南。
