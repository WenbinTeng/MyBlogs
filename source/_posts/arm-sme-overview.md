---
title: arm-sme-overview
date: 2025-08-15 19:12:41
tags:
typora-root-url: ./arm-sme-overview
---

# ARM SME 概述

ARM 可扩展矩阵扩展（Scalable Matrix Extension，SME），是用于增强矩阵运算的指令集扩展。SME 是建立在可扩展向量扩展（Scalable Vector Extension，SVE）上的，新增了处理矩阵的能力。主要特性包括：

- 两个向量之间的外积运算；
- 矩阵分块存储；
- 矩阵分块中向量的 Load/Store/Insert/Extract 操作，以及实时矩阵转置功能；
- 流式 SVE 模式。

下表总结了 SME、SVE、SVE2 的主要特性：

| SME                                       | SVE2                                 | SVE                             |
| ----------------------------------------- | ------------------------------------ | ------------------------------- |
| 流式 SVE 模式                             | NEON DSP ++                          | 可扩展向量                      |
| 实时矩阵转置                              | 多精度算术运算                       | 逐通道谓词执行                  |
| 向量外积运算                              | 匹配检测与直方图统计                 | 聚集加载与分散存储              |
| 矩阵向量的 Load/Store/Insert/Extract 操作 | 非临时性分散/聚集操作                | 推测性向量化                    |
|                                           | 按位转置                             | 机器学习扩展（FP16 + 点积运算） |
|                                           | 高级 SIMD 扩展、SHA3、SM4 加密指令集 | v8.6 BF16、浮点和 Int8 矩阵乘法 |

本博客的编写参考了基于以下 ARM 社区文章：
- [Part 1: Arm Scalable Matrix Extension (SME) Introduction](https://community.arm.com/arm-community-blogs/b/architectures-and-processors-blog/posts/arm-scalable-matrix-extension-introduction)
- [Part 2: Arm Scalable Matrix Extension (SME) Instructions](https://community.arm.com/arm-community-blogs/b/architectures-and-processors-blog/posts/arm-scalable-matrix-extension-introduction-p2)
- [Part 3: Matrix-matrix multiplication. Neon, SVE, and SME compared](https://community.arm.com/arm-community-blogs/b/architectures-and-processors-blog/posts/matrix-matrix-multiplication-neon-sve-and-sme-compared)

### 1. 流式 SVE 模式

SME 引入了流式 SVE 模式，该模式实现了 SVE2 指令集的子集，并新增了 SME 专属指令。流式 SVE 模式对大型数据集进行高吞吐量的“流式”数据处理，这类流式数据通常具有简单的循环控制流和有限的条件分支。而非流式 SVE 模式支持完整的 SVE2 指令集，通用代码通常用于处理复杂数据结构和复杂谓词操作。

大多数 SME 指令仅在流式 SVE 模式下可用。流式向量长度（Streaming Vector Length，SVL）通常会大于非流式向量长度（Non-streaming vector Length，NSVL）。例如，在非流式 SVE 模式下 NSVL 可能是 128 位，而在流式 SVE 模式下 SVL 可能是 512 位。有效的 SVL 在 EL1、EL2 和 EL3 异常级别下由 SMCR_ELx.LEN 寄存器控制。

<img src="1.png" width="70%">

### 2. 非流式 SVE 模式与流式 SVE 模式之间的切换

如果处理器同时实现了 SME 与 SVE2，应用程序可根据需求在不同运行模式间切换。例如，根据需要处理的向量长度，选择不同的处理模式，提升数据处理效率。

SME 新增了 **`PSTATE.{SM, ZA}`** 位用于分别控制启用/禁用流式 SVE 模式和 SME ZA 存储区，可以使用 `MSR` 指令在流式向量控制寄存器（Streaming Vector Control Register，SVCR）中进行修改，具体操作如下：

- `MSR SVCRSM, #<imm>`
- `MSR SVCRZA, #<imm>`
- `MSR SVCRSMZA, #<imm>`

**`SMSTART`** 指令是 `MSR` 指令的别名，可用于设置 PSTATE.SM 和 PSTATE.ZA 的状态：

- `SMSTART`：同时启用流式 SVE 模式和 ZA 存储访问；
- `SMSTART SM`：启用流式 SVE 模式；
- `SMSTART ZA`：启用 ZA 存储访问。

**`SMSTOP`** 指令是 `MSR` 指令的别名，可清除 PSTATE.SM 和 PSTATE.ZA 状态：

- `SMSTOP`：同时禁用流式 SVE 模式和 ZA 存储访问；
- `SMSTOP SM`：禁用流式 SVE 模式；
- `SMSTOP ZA`：禁用 ZA 存储访问。

<img src="2.png" width="30%">

### 3. SME 架构状态

与 SVE2 类似，流式 SVE 模式提供大小为 SVL 的向量寄存器 Z0-Z31 和谓词寄存器 P0-P15。SVE 向量寄存器 Zn 的最低编号位也包含固定长度的 Vn、Qn、Dn、Sn、Hn 和 Bn 寄存器。当进入流式 SVE 模式（PSTATE.SM 从 0 变为 1）或退出流式 SVE 模式（PSTATE.SM 从 1 变为 0）时，所有这些寄存器都会被清零。

<img src="3.png" width="40%">

大多数非流式 SVE2 指令可在流式 SVE 模式下使用，但可能使用不同的向量长度。当前有效向量长度 VL 可通过 RDSVL 指令读取：

- `RDSVL <Xd>, #<imm>`

请注意，在流式 SVE 模式下，软件很少需要显式读取 SVL 值。RDSVL 指令通常用于在非流式模式下确定 SVL 值。

### 4. ZA 矩阵

SME 新引入的 ZA 矩阵是一个二维正方形字节数组，其维度为 `(SVL in bytes) x (SVL in bytes)`。例如，若流式 SVE 模式下的向量长度为 256 位（32 字节），则 ZA 存储的大小为 32x32=1024 字节。

ZA 矩阵可以通过以下三个方式进行访问：

- ZA 矩阵向量
- ZA 分块
- ZA 分块切片

<img src="4.png" width="40%">

##### 4.1. ZA 矩阵向量

就像前面提到的，ZA 矩阵的维度为 SVL 对应的字节数。对每一个 SVL 位的向量，可以按照 8 位、16 位、32 位、64 位或 128 位的方式进行访问：

- ` ZA.B[N], ZA.H[N], ZA.S[N], ZA.D[N], ZA.Q[N]`

为实现上下文切换功能，新增了 SME 架构的 LDR 和 STR 指令，用于在内存与 ZA 矩阵向量之间加载和存储数据：

- `LDR ZA[<Wv>, <imm>], [<Xn|SP>{, #<imm>, MUL VL}]`
- `STR ZA[<Wv>, <imm>], [<Xn|SP>{, #<imm>, MUL VL}]`

##### 4.2. ZA 分块

ZA 矩阵分块是 ZA 矩阵内部的一个二维正方形子矩阵。ZA 矩阵分块的宽度始终为 SVL，与 ZA 矩阵的宽度相同。可用分块数量由元素数据类型大小决定：

- 当元素数据类型为 8 位时，ZA 只能作为一个分块 ZA0.B 进行访问；
- 当元素数据类型为 16 位时，ZA 可被访问为两个分块 ZA0.H-ZA1.H；
- 当元素数据类型为 32 位时，ZA 可被访问为四个分块 ZA0.S-ZA3.S；
- 当元素数据类型为 64 位时，ZA 可被访问为八个分块 ZA0.D-ZA7.D；
- 当元素数据类型为 128 位时，ZA 可被访问为十六个分块 ZA0.Q-ZA15.Q。

例如，若 SVL 为 256 位（32 字节）且元素数据类型大小为 8 位，则 ZA 可视为 ZA0.B，即 32 个（32×1 字节）向量。

<img src="5.png" width="40%">

若 SVL 为 256 位（32 字节）且元素数据类型大小为 16 位，则 ZA 可视为两个分块 ZA0.H 和 ZA1.H，每个分块可视为 16 个（16×2 字节）向量。

<img src="6.png" width="40%">

（*这里我没有理解，为什么数据位宽越大，可访问的矩阵分块数量越多？*）

##### 4.3. ZA 分块切片

ZA 矩阵分块可以进行切片访问。ZA 分块切片是指 ZA 分块内水平或垂直方向上连续元素的一维集合。对 ZA 分块的向量操作实际上就是ZA 分块切片：

- 方向：水平或垂直分块切片通过在 ZA 分块名称后添加 H 或 V 后缀表示；
- 索引：分块切片通过在 ZA 分块名称后添加切片索引 [N] 表示。

例如，下图展示了当 SVL 为 128 位时的 ZA 分块 ZA0.B。ZA0V.B[0]和 ZA0V.B[13]访问垂直分块切片，ZA0H.B[0]和 ZA0H.B[15]访问水平分块切片。

<img src="7.png" width="40%">

下图展示了另一个图块切片访问的示例，其中 SVL 为 128 位，元素类型大小为 16 位：

<img src="8.png" width="40%">

为提高硬件访问 ZA 矩阵分块及其切片时的效率，ZA 矩阵分块的切片采用了交错排列方式。下图展示了这种交错排列的示例。本例中 SVL 为 256 位，元素数据类型大小为 16 位。这意味着 ZA 可视为由 ZA0H 和 ZA1H 两个矩阵分块组成，其水平切片呈交错排列：

<img src="9.png" width="100%">

下图展示了不同元素数据类型大小与水平和垂直分块切片组合视图的示例。左侧列展示了 ZA 存储中每行数据的不同寻址方式。设 `SIZE` 为向量元素的大小，其中对于数据类型 B、H、S、D 或 Q，`SIZE` 分别为 1、2、4、8 或 16。设 `NUM_OF_ELEMENTS` 为向量中的元素数量，即 `bytes_of(SVL)/SIZE`。`ZAnH.<B|H|S|D|Q>[m]` 访问包含 ZA 存储中完整行 `(m*SIZE+n)` 的向量。`ZAnV.<B|H|S|D|Q>[m]` 访问包含列 `(m*SIZE)` 与行 `(i*SIZE+n)` 元素的向量，其中 `i` 为 `0~(NUM_OF_ELEMENTS-1)`。

<img src="10.png" width="100%">
