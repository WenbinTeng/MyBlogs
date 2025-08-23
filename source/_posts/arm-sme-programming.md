---
title: arm-sme-programming
date: 2025-08-22 19:32:14
tags:
typora-root-url: ./arm-sme-programming
---

# ARM SME 编程

SME 提供了众多指令。这些指令能够执行多种多样的操作和计算任务。运用这些指令可以有效处理大规模数据集。

### 1. SME 指令

根据指令功能，可以将 SME 指令分为两类：算术指令和数据传输指令。

##### 1.1. 算术指令

**ADDHA**：该指令将水平向量累加到 ZA 分块中，每个元素由一对谓词进行控制。下图展示了指令 `ADDHA, ZA0.S, P0/M, P1/M, Z1/S` 的执行结果。

<img src="1.png" width="50%">

**ADDVA**：该指令将垂直向量累加到 ZA 分块中，每个元素由一对谓词进行控制。下图展示了指令 `ADDVA、ZA0.S、P0/M、P1/M、Z1/S` 的执行结果。

<img src="2.png" width="50%">

**FMOPA**：浮点外积累加运算。该指令生成第一源向量与第二源向量的外积。对于半精度变体，第一源为 SVLH×1 向量，第二源为 1×SVLH 向量。下图展示了指令 `FMOPA, ZA0.S, P1/M, P2/M, Z1.S, Z2.S` 的执行结果。

<img src="3.png" width="40%">

**SMOPA**：有符号整数外积累加运算。计算第一源向量和第二源向量的外积所生成的子矩阵的有符号累加值。

**UMOPA**：无符号整数外积累加运算。计算第一源向量和第二源向量的外积所生成的子矩阵的无符号累加值。

##### 1.2 数据传输指令

**LD1W**：将内存中连续的 32 位值加载到 ZA 分片切片中。其他加载指令还有： **LD1D**、 **LD1H** 、**LD1S** 等。下图展示了指令 `LD1W, ZA0H.S[W12,#0], P0/Z, [X0]` 的执行结果。

<img src="4.png" width="40%">

**LDR**：加载 ZA 矩阵向量。ZA 矩阵向量由选择的向量寄存器与立即偏移量之和（Streaming SVE 模式中向量字节数）选定。

**ST1B**：从 8 位元素 ZA 分块切片中连续存储字节。切片编号由切片索引寄存器与立即偏移量求和、对向量中 8 位元素数量取模后选定。其他存储指令还有：**ST1D**、**ST1W**、**ST1H ** 等。

**MOV**：将 ZA 分块切片移动到向量寄存器。

### 2. SME 编译选项

在程序中集成 SME 特性一般有两种方式：

- 编写汇编代码，使用Arm 编译器或者 Clang 编译器直接生成可执行文件。
- 使用编译器支持的 SME intrisics 编写代码，编译器会直接使用相应的 SME 指令替换内联函数。

在编译 SME 程序时，需要根据不同的变体选择 `-march` 选项：

| 功能标识符      | 功能描述                              | -march/-mcpu 选项 |
| --------------- | ------------------------------------- | ----------------- |
| FEAT_SME        | SME                                   | sme               |
| FEAT_SME2       | SME2                                  | sme2              |
| FEAT_SME_F64F64 | SME with the double-precision variant | sme-f64f64        |
| FEAT_SME_I16I64 | SME with 16-bit integer variant       | sme-i16i64        |
| ...             | ...                                   | ...               |

例如：

```bash
// Arm C/C++ compiler:
armclang --target=aarch64-arm-none-eabi -march=armv9.2-a+sme2 -o <Executable file> <source file>

// Clang compiler:
clang -target aarch64-none-elf -march=armv9.2-a+sme2 <Source files> -o <Executable files>
```

### 3. SME 调用约定

在使用汇编语言编程 SME 时，需要参考支持 SME 的 Arm 架构应用二进制接口（ABI）文档中 Arm 64 位架构过程调用标准（AAPCS64）。

根据 AAPCS64 标准，前 8 个寄存器 `r0-r7` 用于向子程序传递参数值。在此情况下，`x0-x5` 传递输入矩阵的行列数值及矩阵指针。随后，在矩阵乘法的计算过程中使用这些信息。寄存器 `r0-r7` 同样用于从函数返回结果。除 r0-r7 外，AAPCS64 还定义了其他通用寄存器的使用，并总结了过程调用标准中通用寄存器的用途。此外，SME 定义了若干处理器状态： `ZA storage` 、 `PSTATE.SM` 、 `PSTATE.ZA` 和 `TPIDR2_EL0` 。AAPCS64 描述了跨函数边界的处理器状态管理。

##### 3.1 流模式的使用

ABI 定义了函数入口和正常返回时 `PSTATE.SM` 的状态。处理器在任何时刻都处于流模式（ `PSTATE.SM==1` ）或非流模式（ `PSTATE.SM==0` ）之一。

在直接编写汇编代码时，使用指令 `SMSTART SM` 进入流模式。使用 `SMSTOP SM` 返回到非流模式。在使用 ACLE 内部函数编写 C/C++ 代码时，ACLE 提供了两个属性来指定程序执行语句的方式，分别是流模式（Streaming）和流兼容（Streaming-compatible）模式。

##### 3.2 ZA 存储的使用

ZA 存储的启用由处理器状态位 `PSTATE.ZA` 控制。与流模式类似，启用 ZA 存储有两种方式。第一种是通过直接在汇编中编写的 `SMSTART` 指令。第二种是使用带有 ACLE 内部函数的属性。在 C 和 C++代码中，对 ZA 的访问控制在函数粒度级别。AAPCS 为函数处理 ZA 提供了三种选择：

- 默认情况下，函数不使用 ZA 存储。
- 函数使用 ZA，并与调用者共享。
- 该函数使用其自行创建的 ZA 存储，并且不与调用者共享。

### 4. SME 编程示例

下面将通过两个示例来说明使用汇编语言与 C 语言代码实现两个 4 元素向量乘法，生成一个 4*4 矩阵。

##### 4.1 汇编代码

```assembly
    .global op
    .p2align 2
    .type op, "function"

 op:
    smstart
    PTRUE P0.S

    LD1W   {Z0.S}, P0/Z, [X0]

    LD1W   {Z4.S}, P0/Z, [X1]

    FMOPA ZA0.S, P0/M, P0/M, Z0.S, Z4.S

    mov w12, #0
    ST1W ZA0H.S[W12, #0], P0, [X2]
    mov w11, #4
    ST1W ZA0H.S[W12, #1], P0, [X2, X11, LSL #2]
    mov w11, #8
    ST1W ZA0H.S[W12, #2], P0, [X2, X11, LSL #2]
    mov w11, #12
    ST1W ZA0H.S[W12, #3], P0, [X2, X11, LSL #2]

    smstop
    ret
```

下面逐行解释上述汇编代码中使用的汇编指令：

1. `smstart` ：启用 ZA 存储矩阵并进入流式 SVE 模式。SME 指令只能在流式 SVE 模式下执行。
2. `PTRUE P0.S` ：初始化谓词寄存器 P0。由于本示例未使用谓词化，故将其设置为全真。我们需要初始化一个谓词寄存器，以满足后续 SME 指令的需要。
3. `LD1W {Z0.S}, P0/Z, [X0]` ：将第一个 4 元素输入向量加载到单个向量寄存器中。其中：
   - `Z0.S` ：目标寄存器的名称， `Z0` 。其中 `.S` 表示该寄存器包含 32 位元素。
   - `P0/Z` ：谓词寄存器的名称， `P0/Z` 表示非激活元素被清零，但在本例中所有元素始终处于激活状态。
   - `X0` ：内存中第一个输入向量 A 的地址。 `X0` 、 `X1` 、 `X2` 分别代表向量 A、向量 B 和结果矩阵。
4. `FMOPA ZA0.S, P0/M, P0/M, Z0.S, Z4.S` ：该指令计算第一个源向量与第二个源向量的外积：
   - `ZA0.S` ：结果矩阵存储的 ZA 分块名称，ZA0.S 表示其包含 32 位元素。
   - `P0/M` ：谓词寄存器名称。与 LD1W 指令相同，该谓词寄存器被设置为全真。
   - `Z0.S` 和 `Z4.S` ：包含输入向量的两个向量寄存器，由之前的 `LD1W` 指令加载。这些是 `FMOPA` 指令执行操作时使用的源向量寄存器。该操作计算由 `LD1W` 指令加载的两个源向量寄存器的外积。
5. `mov w12, #0` ：为每条 ST1W 指令设置地址偏移量。其中：
   - `w12` 是包含偏移量的寄存器。
   - `#0` 是一个表示偏移量的整数值。对于第一条指令，偏移量为零，因此向量元素被存储到地址 `X2 + 0` ，即 `X2` 。对于后续的 `ST1W` 指令，偏移量每次增加 4。
6. `ST1W ZA0H.S[W12, #1], P0, [X2, X11, LSL #2]` ：该指令将结果矩阵的单个行存储在分块 `ZA0` 中，并按顺序将元素值存储到内存中。 `ST1W` 指令可以垂直或水平访问分块值。在此情况下，采用水平方式访问。
   - `ZA0H.S` ：与 `LD1W` 指令类似， `ZA0` 表示 ZA 瓦片的名称， `.S` 指定元素为 32 位。该操作数中的 `H` 指定了在瓦片内水平访问值。也就是说，我们以行优先格式而非列优先格式存储结果矩阵。
   - `W12, #1` ：共同指定了正在存储的行号。在此示例中， `W12` 始终为 0，因此这指定了第 1 行。
   - `P0` ：是谓词寄存器，在此示例中设置为全真。
   - `X2` ：结果矩阵在内存中存储的基地址。
   - `X11, LSL #2` ：共同指定一个偏移量，该偏移量与基地址 `X2` 相加，得到该指令要存储的第一个元素的内存地址。第一条 `ST1W` 指令使用的偏移量为 0，随后的 `ST1W` 指令将偏移量每次增加 4。 `LSL #2` 将寄存器值的位左移 2 位，相当于将值乘以 4。
7. `smstop` ：退出 SVE 流模式。
8. `ret` ：从子程序返回。

总结一下，矩阵乘法的一般流程为：使用 `LD1W` 将输入数组值加载到两个向量寄存器中，然后使用 `FMOPA` 计算两个向量寄存器外积的结果矩阵并存入 ZA 分块，最后使用 `ST1W` 将结果矩阵存储到内存中。

##### 4.2 C 代码

```c
#include <stdio.h>
#include <stdlib.h>
#include <inttypes.h>
#include <math.h>
#include <arm_sve.h>

extern void op( const float_t * restrict ,  const float_t * restrict, const float_t * restrict );

float_t main(int argc, char **argv)
{
    int y;

    float_t * vec_a = (float_t *) malloc(4*sizeof(float_t));
    float_t * vec_b = (float_t *) malloc(4*sizeof(float_t));
    float_t * result_matrix = (float_t *) malloc(16*sizeof(float_t));

    vec_a[0] = 7.0;
    vec_a[1] = 3.0;
    vec_a[2] = 6.0;
    vec_a[3] = 9.0;

    vec_b[0] = 4.0;
    vec_b[1] = 2.0;
    vec_b[2] = 1.0;
    vec_b[3] = 5.0;

    for (y=0; y<16; y++) {
        result_matrix[y] = 0;
    }

    op(&vec_a[0], &vec_b[0], &result_matrix[0]);

    printf("Vector A:\n");
    for (y=0; y<4; y++) {
                printf("  %4.0f\n", vec_a[y]);
    }

    printf("Vector B:\n");
    for (y=0; y<4; y++) {
                printf("  %4.0f\n", vec_b[y]);
    }

    printf("Result matrix:\n");
    for (y=0; y<16; y+=4) {
                printf("%4.0f  %4.0f  %4.0f  %4.0f\n", result_matrix[y], result_matrix[y+1], result_matrix[y+2], result_matrix[y+3]);
    }

    return 0;
}
```

上述 C 代码执行了以下三个操作：

1. 定义并初始化三个数组：
   - `Vec_a` 和 `vec_b` 是输入向量。每个向量都是一个包含 4 个浮点数值的数组。
   - `Result_matrix` 是结果矩阵，初始化为包含 16 个值的数组，所有值均初始化为零。
2. 调用汇编函数 `op` ，传入 `vec_a` 、 `vec_b` 和 `result_matrix` 的内存地址。ARM ABI 规定这些参数分别通过寄存器 `X0` 、 `X1` 和 `X2` 传递。
3. 使用 `printf` 将两个向量及结果矩阵的值输出到控制台。





