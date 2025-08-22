---
title: arm-neon-programming
date: 2025-08-22 19:31:28
tags:
---

# ARM NEON 编程

GNU 工具和 RealView 编译工具（RVCT）的最新版本均支持 NEON 指令。

### 1. NEON Assembler

使用 NEON 单元最直接的方式是编写汇编代码。GNU 和 RVCT 汇编器采用相同的指令格式，但是存在一些差异，包括：汇编器指示（assembler directives）、标签格式（format of labels）、注释指示符（comment indicators）。

以下示例展示了使用 GNU 汇编器（Gas）执行 NEON 指令的汇编函数：

```assembly
    .text
    .arm
    .global double_elements
double_elements:
    vadd.i32 q0,q0,q0
    bx       lr
    .end
```

要使用 Gas 汇编上述示例中的代码，需要在汇编器命令行中添加 `-mfpu=neon` ，例如：`arm-none-linux-gnueabi-as -mfpu=neon asm.s`。

以下示例展示了相同功能的 RVCT 格式。

```assembly
    AREA RO, CODE, READONLY
    ARM
    EXPORT double_elements
double_elements
    VADD.I32 Q0, Q0, Q0
    BX       LR
    END
```

要使用 RVCT 汇编上述示例中的代码，必须指定支持 NEON 指令的目标处理器，例如：`armasm --cpu=Cortex-A8 asm.s`。

### 2. NEON Intrisics

NEON Intrisics （内联函数与数据类型）提供了与内联汇编相似的功能，并具备类型检查和自动寄存器分配等附加特性。内联函数在 C 或 C++中呈现为函数调用形式，但在编译期间会被替换为底层指令序列。这意味着可以用高级语言来表达底层的架构行为。

除了让程序员直接访问那些通常难以映射到高级语言语句的指令外，使用内联函数还意味着编译器可以对操作进行优化以提升性能。开发者无需考虑寄存器分配和互锁问题，因为编译器会处理这些事务。

GCC 和 RVCT 支持相同的 NEON 内联函数语法，使得 C 或 C++代码能在不同工具链间移植。要添加 NEON 内联函数支持，需包含头文件 `arm_neon.h` 。以下示例通过 C 代码中的内联函数（而非汇编指令）实现了与汇编示例相同的功能。

```c
#include <arm_neon.h>

uint32x4_t double_elements(uint32x4_t input)
{
    return(vaddq_u32(input, input));
}
```

要在 GCC 中使用 NEON intrinsics，必须在编译器命令行中指定 `-mfpu=neon` ：`arm-none-linux-gnueabi-gcc -mfpu=neon intrinsic.c`。根据使用的工具链，可能还需要添加 `-mfloat-abi=softfp` 来向编译器表明 NEON 变量必须通过通用寄存器传递。

RVCT 编译器支持 NEON intrinsics，前提是在编译器命令行中指定了支持 NEON 指令的目标处理器。例如：`armcc --cpu=Cortex-A9 intrinsic.c`。

### 3. 自动向量化

编译器还能对 C 或 C++源代码执行自动向量化。这样无需编写汇编代码或使用内部函数即可获得高性能 NEON 支持，同时保持源代码在不同工具和目标平台间的可移植性。由于 C 语言未规定并行化行为，可能需要向编译器额外提示哪些操作可安全且最优地并行化。这一过程不会影响源代码在不同平台或工具链之间的可移植性。

以下示例展示了一个编译器可安全且高效实现向量化的小型函数。之所以能实现这一点，是因为程序员使用 `__restrict` 关键字确保指针 `pa` 和 `pb` 不会指向内存中的重叠区域。程序员还通过屏蔽 `n` 的低两位进行边界测试，强制 for 循环始终以四的倍数执行。这些额外信息使编译器能够安全地将该函数向量化为 NEON 加载和存储操作。

```c
void add_ints(int * __restrict pa, int * __restrict pb, unsigned int n, int x)
{
    unsigned int i;

    for(i = 0; i < (n & ~3); i++)
        pa[i] = pb[i] + x;
}
```

##### 3.1 使用 GCC 进行自动向量化

要启用自动向量化，您必须在 GCC 命令行中添加 `-mfpu=neon` 和 `-ftree-vectorize` 。例如：`arm-none-linux-gnueabi-gcc -mfpu=neon -ftree-vectorize -c vectorized.c`。

根据您使用的工具链，可能还需要添加 `-mfloat-abi=softfp` 来指定 NEON 变量必须通过通用寄存器传递。可以通过在命令行中添加 `-ftree-vectorizer-verbose=1` 来请求更详细的编译器输出。这将使编译器输出已完成向量化处理的代码、编译器无法向量化的代码以及未能实现向量化的原因提示。可以利用这些信息将代码修改为编译器可向量化的格式。

##### 3.2 使用 RVCT 进行自动向量化

要启用自动向量化，必须指定包含 NEON 技术的目标处理器，采用 `-O2` 或更高级别的优化进行编译，并在命令行中添加 `-Otime and --vectorize` 参数。例如：`armcc --cpu=Cortex-A9 -O3 -Otime --vectorize -c vectorized.c`。当指定 `--vectorize` 时，只有在同时指定 `-Otime` 且优化级别为 `-O2` 或 `-O3` 时，才会启用自动向量化。由于浮点值的并行累加可能降低通过排序输入数据获得的精度，除非在命令行中指定 `--fpmode=fast` ，否则这些操作将被禁用。

可以通过在命令行中添加 `--remarks` 来请求更详细的编译器输出，这将提供有关编译过程中多个方面的额外信息，包括已完成向量化处理的代码、编译器无法向量化的代码以及未能实现向量化的原因提示。这些信息可用于将代码修改为编译器能够向量化的格式。

### 4. 使用 NEON 优化的库

在系统中使用 NEON 技术最简单的方式，就是直接调用针对 NEON 优化的函数库。

OpenMAX 是由 Khronos 集团创建并发布的免版税跨平台 API 标准。ARM 公司开发了基于 ARMv7 NEON 优化的 OpenMAX 开发层（DL）实现版本，可以在 ARM 官网下载该组件。

以下示例通过调用 OpenMAX 函数 `omxSP_DotProd_S16()` 计算两个有符号 16 位整数向量的点积。当使用 ARMv7 优化版 OpenMAX DL 库时，该函数通过 NEON 向量运算实现。

```c
#include <omxSP.h>

OMX_S16 source1[] = {42, 23, 983, 7456, 124, 11111, 4554, 10002};
OMX_S16 source2[] = {242, 423, 9832, 746, 1124, 1411, 2254, 1298};
OMX_S32 source_dotproduct(void)
{
    OMX_INT len = sizeof(source1)/sizeof(OMX_S16);
    
    return omxSP_DotProd_S16(source1, source2, len);
}
```
