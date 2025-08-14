---
title: arm-overview
date: 2025-08-14 22:51:53
tags:
---

# ARM 架构概览

ARM，即 Advanced RISC Machines，是围绕精简指令集构建的一套处理器生态系统，包括指令集架构、处理器产品、嵌入式系统设计规范等内容。

### 1. ARM 指令集架构与处理器型号的命名

许多刚接触 ARM 的朋友可能会困扰于它们的指令集与处理器的命名，现在我们先来缕清其命名规则。

##### 1.1 ARM 指令集命名规则


```
ARMv<n><variants>x<variants>
```

其中：
- `<n>`：表示版本号，目前为 1-9；
- `<variants>`：表示变种，指的是在标准系统基础上新增的功能：
  - `T`：Thumb 指令集支持；
  - `M`：长乘法指令支持；
  - `E`：增强型 DSP 指令支持；
  - `J`：Java 加速器 Jazelle；
  - `SIMD`：ARM 多媒体功能支持。
- `x`：表示缺少变种的支持。

以上变体在 ARMv6 指令集中都默认支持，因此不再单独列出。从 ARMv7 指令集开始，ARM 公司将处理器产品线划分为三个部分：A 系列，面向性能密集型系统的应用处理器内核；R 系列，面向实时应用的高性能内核；M 系列，面向各类嵌入式应用的微控制器内核。这三个系列也对应于 ARM 三个字母。这三类产品所使用的指令集对应为：ARMv7-A、ARMv7-R、ARMv7-M。

##### 1.2 ARM 处理器命名规则

**ARMv1-2 时期**
该时期没有对应的 ARM 处理器产品。

**ARMv3-6 时期**
该时期的 ARM 处理器产品的命名规则为：

```
ARM<x><y><z><T><D><M><I><E><J><F><-S>
```

其中：
- `<x>`：处理器系列，7-10；
- `<y>`：存储管理/保护单元数量；
- `<z>`：cache 数量；
- `<T>`：支持 Thumb 指令集；
- `<D>`：支持片上调试；
- `<M>`：支持快速乘法；
- `<I>`：支持 Embedded ICE，支持嵌入式跟踪调试；
- `<E>`：支持增强型 DSP 指令；
- `<J>`：支持 Jazelle；
- `<F>`：具备向量浮点单元 VFP；
- `<-S>`：可综合版本。

**ARMv7 以后时期**
该时期 ARM 处理器产品被划分为三个系列，其命名规则为：

```
Cortex-<series><number>
```

其中：
- `<series>`：产品系列，A/R/M，分别代表高性能处理器、实时处理器、微控制器系列。
- `<number>`：产品型号，代表处理器性能等级，7/5/3，分别代表高级、中级、低级的处理器性能。

##### 1.3 指令集版本与处理器型号对照表

| <div style="width: 80pt">**Architecture**</div> | <div style="width: 20pt">**BW**</div> | <div style="width: 100pt">**Arm Ltd. Cores**</div>           | <div style="width: 120pt">**Third-Party Cores**</div>        | <div style="width: 90pt">**Profile **</div> |
| ----------------------------------------------- | ------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------- |
| ARMv1                                           | 32                                    | ARM1                                                         |                                                              | Classic                                     |
| ARMv2                                           | 32                                    | ARM2<br>ARM250<br>ARM3                                       | Amber<br>STORM Open Soft Core                                | Classic                                     |
| ARMv3                                           | 32                                    | ARM6<br>ARM7                                                 |                                                              | Classic                                     |
| ARMv4                                           | 32                                    | ARM8                                                         | StrongARM<br>FA526<br>ZAP Open Source Processor Core         | Classic                                     |
| ARMv4T                                          | 32                                    | ARM7TDMI<br>ARM9TDMI<br>SecurCore SC100                      |                                                              | Classic                                     |
| ARMv5TE                                         | 32                                    | ARM7EJ<br>ARM9E<br>ARM10E                                    | XScale<br>FA626TE<br>Feroceon<br>PJ1 / Mohawk                | Classic                                     |
| ARMv6                                           | 32                                    | ARM11                                                        |                                                              | Classic                                     |
| ARMv6-M                                         | 32                                    | Cortex-M0<br>Cortex-M0+<br>Cortex-M1<br>SecurCore SC000      |                                                              | Microcontroller                             |
| ARMv7-M                                         | 32                                    | Cortex-M3<br>SecurCore SC300                                 | Apple M7 Motion Coprocessor                                  | Microcontroller                             |
| ARMv7E-M                                        | 32                                    | Cortex-M4<br>Cortex-M7                                       |                                                              | Microcontroller                             |
| ARMv8-M                                         | 32                                    | Cortex-M23<br>Cortex-M33                                     |                                                              | Microcontroller                             |
| ARMv8.1-M                                       | 32                                    | Cortex-M55<br>Cortex-M85                                     |                                                              | Microcontroller                             |
| ARMv7-R                                         | 32                                    | Cortex-R4<br>Cortex-R5<br>Cortex-R7<br>Cortex-R8             |                                                              | Real-Time                                   |
| ARMv8-R                                         | 32                                    | Cortex-R52                                                   |                                                              | Real-Time                                   |
|                                                 | 64                                    | Cortex-R82                                                   |                                                              | Real-Time                                   |
| ARMv7-A                                         | 32                                    | Cortex-A5<br>Cortex-A7<br>Cortex-A8<br>Cortex-A9<br>Cortex-A12<br>Cortex-A15<br>Cortex-A17 | Qualcomm Scorpion / Krait<br>PJ4 / Sheeva<br>Apple Swift (A6, A6X) | Application                                 |
| ARMv8-A                                         | 32                                    | Cortex-A32                                                   |                                                              | Application                                 |
|                                                 | 64/32                                 | Cortex-A35<br>Cortex-A53<br>Cortex-A57<br>Cortex-A72<br>Cortex-A73 | X-Gene<br>Nvidia Denver 1/2<br>Cavium ThunderX<br>AMD K12<br>Apple Cyclone (A7) / Typhoon (A8, A8X) / Twister (A9, A9X) / Hurricane+Zephyr (A10, A10X)<br>Qualcomm Kryo<br>Samsung M1 / M2 ("Mongoose") / M3 ("Meerkat") | Application                                 |
|                                                 | 64                                    | Cortex-A34                                                   |                                                              | Application                                 |
| ARMv8.1-A                                       | 64/32                                 |                                                              | Cavium ThunderX2                                             | Application                                 |
| ARMv8.2-A                                       | 64/32                                 | Cortex-A55<br>Cortex-A75<br>Cortex-A76<br>Cortex-A77<br>Cortex-A78<br>Cortex-X1<br>Neoverse N1 | Nvidia Carmel<br>Samsung M4 ("Cheetah")<br>Fujitsu A64FX (ARMv8 SVE 512-Bit) | Application                                 |
|                                                 | 64                                    | Cortex-A65<br>Neoverse E1 With SMT<br>Cortex-A65AE           | Apple Monsoon+Mistral (A11) (September 2017)                 | Application                                 |
| ARMv8.3-A                                       | 64/32                                 |                                                              |                                                              | Application                                 |
|                                                 | 64                                    |                                                              | Apple Vortex+Tempest (A12, A12X, A12Z)<br>Marvell ThunderX3 (v8.3+) | Application                                 |
| ARMv8.4-A                                       | 64/32                                 |                                                              |                                                              | Application                                 |
|                                                 | 64                                    | Neoverse V1                                                  | Apple Lightning+Thunder (A13)<br>Apple Firestorm+Icestorm (A14, M1) | Application                                 |
| ARMv8.5-A                                       | 64/32                                 |                                                              |                                                              | Application                                 |
|                                                 | 64                                    |                                                              |                                                              | Application                                 |
| ARMv8.6-A                                       | 64                                    |                                                              | Apple Avalanche+Blizzard (A15, M2)<br>Apple Everest+Sawtooth (A16)<br>Apple Coll (A17)<br>Apple Ibiza / Lobos / Palma (M3) | Application                                 |
| ARMv8.7-A                                       | 64                                    |                                                              |                                                              | Application                                 |
| ARMv8.8-A                                       | 64                                    |                                                              |                                                              | Application                                 |
| ARMv8.9-A                                       | 64                                    |                                                              |                                                              | Application                                 |
| ARMv9.0-A                                       | 64                                    | Cortex-A510<br>Cortex-A710<br>Cortex-A715<br>Cortex-X2<br>Cortex-X3<br>Neoverse E2<br>Neoverse N2<br>Neoverse V2 |                                                              | Application                                 |
| ARMv9.1-A                                       | 64                                    |                                                              |                                                              | Application                                 |
| ARMv9.2-A                                       | 64                                    | Cortex-A520<br>Cortex-A720<br>Cortex-X4<br>Neoverse V3<br>Cortex-X925<br>Cortex-A320 | Apple Donan / BravaChop / Brava (Apple M4)<br>Apple Tupai / Tahiti (A18) | Application                                 |

### 2. ARM 处理器运行模式

ARM 处理器有 7 种运行模式。用户模式即正常程序运行时的模式，此时应用程序不能够访问一些受操作系统保护的系统资源。

除了用户模式之外的 6 种处理器模式称为特权模式。特权模式下，系统可以访问所有的系统资源，也可以任意地进行处理器模式的切换。系统模式拥有和用户模式一样的寄存器。通常操作系统的任务需要访问所有的系统资源，同时该任务仍然使用用户模式的寄存器组，而不是使用异常模式下相应的寄存器组，这样可以保证当异常中断发生时任务状态不被破坏。

除了系统模式之外的 5 种特权模式称为异常模式，当应用程序发生异常中断时，处理器进入相应的异常模式。每一种异常模式都有一组寄存器，供相应的异常处理程序使用，这样就可以保证在进入异常模式时，用户模式下的寄存器（保存了程序运行状态）不被破坏。

| 处理器模式                           | 描述                               |
| ------------------------------------ | ---------------------------------- |
| 用户模式（User，usr）                | 正常程序执行的模式                 |
| 快速中断模式（FIQ，fiq）             | 用于高速数据传输和通道处理         |
| 外部中断模式（IRQ，irq）             | 用于通常的中断处理                 |
| 特权模式（Supervisor，sve）          | 供操作系统使用的一种保护模式       |
| 数据访问中止模式（Abort，abt）       | 用于虚拟存储及存储保护             |
| 未定义指令中止模式（Undefined，und） | 用于支持通过软件仿真硬件的协处理器 |
| 系统模式（System，sys）              | 用于运行特权级的操作系统任务       |



### 3. ARM 寄存器架构

ARM 处理器中包括 37 个寄存器，包括 31 个 32 位的通用寄存器与 6 个 32 位的状态寄存器。ARM 处理器一共有 7 中不同的处理器模式，每一种处理器模式中有一组相应的寄存器组。任意模式下，可见的寄存器包括 15 个通用寄存器（R0-R14）、1-2 个状态寄存器、1 个程序计数器（PC）。在所有的寄存器中，有些是各模式共用的同一个物理寄存器；有一些寄存器是各模式自己拥有的独立的物理寄存器。

| 用户模式 | 系统模式 | 特权模式 | 中止模式 | 未定义指令模式 | 外部中断模式 | 快速中断模式 |
| -------- | -------- | -------- | -------- | -------------- | ------------ | ------------ |
| R0       | R0       | R0       | R0       | R0             | R0           | R0           |
| R1       | R1       | R1       | R1       | R1             | R1           | R1           |
| R3       | R3       | R3       | R3       | R3             | R3           | R3           |
| R4       | R4       | R4       | R4       | R4             | R4           | R4           |
| R5       | R5       | R5       | R5       | R5             | R5           | R5           |
| R6       | R6       | R6       | R6       | R6             | R6           | R6           |
| R7       | R7       | R7       | R7       | R7             | R7           | R7           |
| R8       | R8       | R8       | R8       | R8             | R8           | R8_fiq       |
| R9       | R9       | R9       | R9       | R9             | R9           | R9_fiq       |
| R10      | R10      | R10      | R10      | R10            | R10          | R10_fiq      |
| R11      | R11      | R11      | R11      | R11            | R11          | R11_fiq      |
| R12      | R12      | R12      | R12      | R12            | R12          | R12_fiq      |
| R13      | R13      | R13_svc  | R13_abt  | R13_und        | R13_irq      | R13_fiq      |
| R14      | R14      | R14_svc  | R14_abt  | R14_und        | R14_irq      | R14_fiq      |
| PC       | PC       | PC       | PC       | PC             | PC           | PC           |
| CPSR     | CPSR     | CPSR     | CPSR     | CPSR           | CPSR         | CPSR         |
|          |          | SPSR_svc | SPSR_abt | SPSR_und       | SPSR_irq     | SPSR_fiq     |

##### 3.1 通用寄存器

通用寄存器分为三类：未备份寄存器（Unbanked Registers）R0-R7、备份寄存器（Banked Registers）R8-R14、程序计数器（Program Counter，PC）R15。

**未备份寄存器**

未备份寄存器包括 R0-R7。对于每一个未备份寄存器地址，在所有的处理器模式下指的都是同一个物理寄存器。在异常中断造成处理器模式切换时，由于不同的处理器模式使用相同的物理寄存器，可能造成寄存器中数据被破坏。未备份寄存器没有被系统用于特别的用途，任何可采用通用寄存器的应用场合都可以使用未备份寄存器。

**备份寄存器**

备份寄存器包括 R8-R14。对于 R8-R12 来说，每个寄存器地址对应 2 个物理寄存器，分别是 Rx 与 Rx_fiq。后者用于快速中断模式下的中断处理，FIQ处理程序可以不必执行保存和恢复中断现场的指令，从而可以使中断处理过程非常迅速。

对于 R13-R14 来说，每个寄存器地址对应 6 个物理寄存器，Rx 是用户模式和系统模式共用的，Rx_svc/Rx_abt/Rx_abt/Rx_und/Rx_irq/Rx_fiq 对应于其他 5 种处理器模式。

R13 常被用作栈指针。应用程序初始化 R13，使其指向该异常模式专用的栈地址。当进入异常模式时，将需要使用的寄存器值保存在 R13 所指的栈中；当退出异常处理程序时，将保存在 R13 所指的栈中的寄存器值弹出。这样就使异常处理程序不会破坏被其中断程序的运行现场。

R14 又被称为链接寄存器，具有两种作用：（1）用于存放每种处理器模式下子程序返回地址；（2）用于存放异常中断发生时的异常模式返回地址。

**程序计数器**

R15 为程序计数器，用于保存当前指令的内存地址。

##### 3.2 程序状态寄存器

当前程序状态寄存器（Current Program State Register，CPSR）可以在任何处理器模式下被访问。它包含了条件标志位、中断禁止位、当前处理器模式标志以及其他的一些控制和状态位。保存程序状态寄存器（Saved Program State Register，SPSR）用于存放 CPSR 寄存器的内容，以便异常返回后恢复异常发生时的工作状态。用户模式和系统模式不是异常中断模式，没有 SPSR。

| 31   | 30   | 29   | 28   | 27   | 26 ... 8 | 7    | 6    | 5    | 4  3  2  1  0 |
| ---- | ---- | ---- | ---- | ---- | -------- | ---- | ---- | ---- | ------------- |
| N    | Z    | C    | V    | Q    |          | I    | F    | T    | M             |

**条件标志位**

- N（Negative flag，负数标志）：第 31 位
  - 当指令执行的结果为负数时，N 标志置 1；否则为 0。
  - 例如，在减法操作中，如果结果为负数，N 标志位将被设置为 1。
- Z（Zero flag，零标志）：第 30 位
  - 当指令执行的结果为 0 时，Z 标志置 1；否则为 0。
  - 例如，在比较操作中，如果两个操作数相等，Z 标志位将被设置为 1。
- C（Carry flag，进位标志）：第 29 位
  - 用于记录无符号数运算中的进位或借位结果。在加法中，如果有进位，C 标志置 1；在减法中，如果有借位，C 标志位也会被设置。
- V（Overflow flag，溢出标志）：第 28 位
  - 用于记录有符号数运算中的溢出情况。如果操作结果溢出（超出有符号数的表示范围），V 标志置 1；否则为 0。

**中断屏蔽位（Interrupt Mask bits）**

- I（IRQ 中断屏蔽）：第 7 位
  - 当 I 位为 1 时，屏蔽 IRQ（普通中断）；为 0 时，允许 IRQ 中断。
  - 这允许处理器在某些关键代码执行期间禁止普通中断，以避免被打断。
- F（FIQ 中断屏蔽）：第 6 位
  - 当 F 位为 1 时，屏蔽 FIQ（快速中断）；为 0 时，允许 FIQ 中断。
  - FIQ 是一种比 IRQ 更快速的中断类型，通常用于时间敏感的任务。

**Thumb 状态位**

- T（Thumb 状态标志位）：第 5 位
  - 当 T 位为 1 时，处理器运行在 Thumb 状态下，即执行 16 位指令集；当 T 位为 0 时，处理器运行在 ARM 状态下，执行 32 位指令集。
  - Thumb 状态是 ARM 处理器的一种优化模式，用于减少指令的存储空间需求，提高代码密度。

**模式位（Mode bits）**

模式位决定了当前处理器的运行模式。ARM处理器有多种运行模式，用于处理不同的任务，如普通程序执行、异常处理、中断处理等。

模式位定义（M4-M0）：第 4-0 位
- 0b10000：**User（用户模式）** - 普通程序的非特权模式，应用程序在此模式下运行。
- 0b10001：**FIQ（快速中断模式）** - 处理FIQ中断。
- 0b10010：**IRQ（普通中断模式）** - 处理IRQ中断。
- 0b10011：**Supervisor（管理模式）** - 进入异常处理（如复位或系统调用）时的特权模式。
- 0b10111：**Abort（终止模式）** - 处理内存访问出错的异常。
- 0b11011：**Undefined（未定义模式）** - 处理未定义指令异常。
- 0b11111：**System（系统模式）** - 具有特权的用户模式，允许直接访问系统资源。

不同的模式可以控制不同的访问权限和寄存器集合。用户模式是非特权模式，其他模式都是特权模式，允许访问更多的系统资源和寄存器。

### 4. ARM 存储系统

ARM 使用单一地址空间，32 位系统中拥有 2^32 个字节。每个字单元中包含四个字节单元或者两个半字单元；一个半字单元中包含两个字节单元。ARM 默认使用小端模式进行存储，也支持使用大端模式进行存储。如果使用非对齐的地址进行访问，要么会产生不可预测的执行结果，要么由处理器进行地址对齐，要么由存储系统忽略地址低位。

### 5. ARM 异常中断

在 ARM 体系结构中，有 3 种方式控制程序的执行流程：

- 在正常程序执行流程中，每执行一条指令，PC 的值加 4；每执行一条 Thumb 指令，PC 的值加 2。整个过程是顺序执行的。
- 通过跳转指令，程序可以跳转到特定的地址标号处执行，或者跳转到特定的子程序处执行。其中，B 指令用于执行跳转操作；BL指令在执行跳转操作的同时，保存子程序的返回地址；BX 指令在执行跳转操作的同时，根据目标地址的最低位可以将程序状态切换到 Thumb 状态；BLX 指令执行 3 个操作，跳转到目标地址处执行、保存子程序的返回地址、根据目标地址的最低位可以将程序状态切换到 Thumb 状态。
- 当异常中断发生时，系统执行完当前指令后，将跳转到相应的异常中断处理程序处执行。在异常中断处理程序执行完成后，程序返回到发生中断的指令的下一条指令处执行。在进入异常中断处理程序时，要保存被中断的程序的执行现场，在从异常中断处理程序退出时，要恢复被中断的程序的执行现场。

ARM 规定的异常中断如表所示。

| 异常中断类型                          | 含义                                                         |
| ------------------------------------- | ------------------------------------------------------------ |
| 复位（Reset）                         | 当处理器复位引脚有效时，系统产生复位异常中断，程序跳转到复位异常中断处理程序处执行。复位异常在系统加电时、系统复位时、软件复位时进行。 |
| 未定义的指令（Undefined Instruction） | 当前指令不是处理器或协处理器定义的指令时，产生未定义指令异常中断。 |
| 软件中断（Software Interrupt，SWI）   | 由用户定义的中断指令触发。                                   |
| 指令预取中止（Prefetch Abort）        | 预取的指令地址不存在，或者该地址不允许当前指令访问，则产生指令预取中止异常中断。 |
| 数据访问中止（Data Abort）            | 数据访问的地址不存在，或者该地址不允许当前指令访问，则产生数据访问中止异常中断。 |
| 外部中断请求（IRQ）                   | 当处理器外部中断请求引脚有效，而且 CPSR 的 I 控制位为 0，则产生外部中断请求异常中断。 |
| 快速中断请求（FIQ）                   | 当处理器外部快速中断请求引脚有效，而且 CPSR 的 F 控制位为 0，则产生外部快速中断请求异常中断。 |

##### 5.1 异常响应过程

1. 保存处理器当前状态、中断屏蔽位以及各条件标志位：将 CPSR 内容保存到将要执行的异常中断对应的 SPSR 物理寄存器中。
2. 设置处理器执行状态、中断屏蔽位：设置 CPSR 中的模式位，以切换处理器执行模式；设置 IRQ 标志位为 1，以禁止外部中断请求响应；如果需要进入快速中断请求模式，则设置 FIQ 标志位为 1。
3. 将链接寄存器（R14）设置为异常中断返回地址。
4. 将程序计数器（R15）设置为该异常中断的中断向量地址。

##### 5.2 异常返回过程

1. 恢复被中断程序的处理器状态：将当前 SPSR 内容复制到 CPSR 中。
2. 返回到发生异常中断的指令的下一条指令处执行：将链接寄存器（R14）内容复制到程序计数器（R15）。
