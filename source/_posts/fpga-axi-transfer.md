---
title: fpga-axi-transfer
date: 2025-05-02 08:49:06
tags:
typora-root-url: fpga-axi-transfer
---

# AXI 总线学习 (4) - 传输特性

在 AXI4 协议中，为了适应现代 SoC 中复杂且高效的数据传输需求，除了基本的握手机制和通道定义外，还引入了一系列高级传输特性，如突发传输（Burst）、超前传输（Outstanding）、乱序响应（Out-of-Order）、通道交织（Interleaving）、窄传输（Narrow）以及非对齐传输（Unaligned）。本文将介绍这些 AXI4 传输特性。



### 1. Burst

AXI4 支持**突发传输（Burst）**，进行一次地址握手之后可以进行多次数据传输，从而减少总线开销、提高带宽利用率。更具体地说，在 Burst 读操作中，Manager 只需要发送一个读起始地址，Subordinate 就根据这个起始地址与响应的地址增长规则自动进行地址计算，将对应的数据发送给 Manager。类似地，在 Burst 写操作中，Manager 只需要发送一个写起始地址，Subordinate 根据传输的长度将数据传输到对应的缓存中。因此，只需要在地址通道中进行一次握手就可以完成连续的数据传输，避免了系统总线的占用。

<img src="burst_read.png" style="zoom: 80%;" />

<img src="burst_write.png" style="zoom: 80%;" />

在 AXI4 协议中，可以在通道中传输 `ARBURST` 和 `AWBURST` 信号来指定 Burst 的地址增长类型：

- **地址固定**（FIXED，AxBURST=0b00）：会多次访问从设备中相同地址的数据，常用于 FIFO 场景中。
  - 每个数据的地址都是固定的。
  - 一次事务中最多可以进行 16 次传输。
- **地址线性递增**（INCR，AxBURST=0b01）：可以配置固定长度的地址递增和非固定长度的地址递增，常用于 Memory 访问的场景中。
  - 每个数据的地址都是上一个数据的地址加上数据宽度。例如，若 `ARSIZE` （读数据宽度的对数）为 2、`ARLEN` （读数据个数-1）为 3，那么从设备将会返回四个地址的数据：`ARADDR`，`ARADDR+4`，`ARADDR+8`，`ARADDR+12`。
  - 一次事务中的传输次数必须为 1、2、4、8 或 16。
  - 不支持提前结束。
- **地址回绕递增**（WRAP，AxBURST=0b10）：地址按照指定的数据宽度进行递增，当到达一个边界后，从初始地址开始的字节对齐位置继续递增，常用于 Cache 访问的场景中。
  - 每个数据的地址都是上一个数据的地址加上数据宽度，如果达到边界，则从初始地址开始，即 $(len+1) \times 2^{size}$。例如，若 `ARSIZE` （读数据宽度的对数）为 2、`ARLEN` （读数据个数-1）为 3、初始地址 `ARADDR` 为 0x000C。那么传输长度为 16，对齐地址为 0x0000，一共需要传输 4 个地址的数据：`0x000C`，`0x0000`，`0x0004`，`0x0008`。
  - 初始地址必须与每次传输的大小对齐。
  - 一次事务中的传输次数必须为 1、2、4、8 或 16。
  - 不支持提前结束。

**优点**：提升单个事务传输的效率，释放地址通道。



### 2. Outstanding

AXI4 支持多个未完成的事务同时进行**超前传输（Outstanding）**，即 Manager 可以在一个读事务或写事务未完成时，就继续发起新的事务，减少 Manager 的等待。这是因为 AXI 通道是分离的，Manager 无需等待数据通道和响应通道中的传输完成，就可以在地址通道中进行新的事务的握手。简单而言，地址通道、数据通道与响应通道之间可以通过 Outstanding 传输模式达到流水线并行。通道中的每个事务通过 ID 信号（ARID/AWID）进行标识，Subordinate 根据自身的能力处理通道中的多个事务。

<img src="outstanding_read.png" style="zoom: 80%;" />

<img src="outstanding_write.png" style="zoom: 80%;" />

注意，在系统设计上要注意 Manager 侧的读缓存与 Subordinate 侧的写缓存大小需求，避免出现无法处理大量数据而出现的系统反压。

**优点**：主设备可以连续发出多个请求，而无需等待每个响应返回，提升带宽利用率。



### 3. Out-of-Order

AXI4 的默认响应是顺序的，但协议允许 Subordinates 进行**乱序响应（Out-of-Order）**，只要每个事务的 ID 匹配。在一般情况下，在读事务中，返回的读数据的 RID 需与相应读地址的 ARID 一致；在写事务中，写数据的 WID 及写响应的 BID 需与相应写地址的 AWID 一致。由于有了事务的 ID 进行对照，我们就可以允许多个从设备乱序地返回事务的数据或响应。访问速度快的从设备可以提前在通道中返回数据和响应，而访问速度慢的从设备可以后续再返回，这样可以充分利用总线的带宽进行传输。

<img src="ooo.png" style="zoom: 80%;" />

注意，如果主设备需要保证事务之间的返回顺序，则应该使用同一个 ID 进行访问，通过地址来区分不同的从设备。

**优点**：可以匹配不同速度的从设备，充分利用带宽。



### 4. Interleaving

AXI4 允许同一 Manager 与 Subordinate 之间的多个突发事务在数据层面进行**交织传输（Interleave）**，只要它们使用不同的 ID。在一个 Burst 传输未完成时，可以插入另一个 ID 的 Burst，有效避免总线空转。

<img src="interleaving.png" style="zoom: 80%;" />

注意，同一 ID 之内的数据还是顺序返回的。是否支持读交织只与从设备的设计有关。个人理解，乱序响应与交织传输的区别是，它们分别是事务（Transaction）层面的乱序与传输（Transfer）层面的乱序。

**优点**：减少事务内传输的空闲等待，能更细粒度地利用带宽进行传输。



### 5. Narrow

AXI4 支持所谓的 **Narrow transfers**，即 Manager 访问的数据宽度小于 Subordinate 返回的数据总线宽度，主要用于传输带宽有限的主设备，或是访问特定寄存器的场景。在 AXI4 协议中可以通过 `strb` 字段指定有效的数据传输位宽，从设备会自动根据 valid byte lanes 进行掩码处理。例如，小位宽的主设备可以变换 `strb` 信号来逐段写入大位宽从设备的一行。



### 6. Unaligned

AXI4 还支持 **Unaligned transfers**，即访问地址不是数据宽度的整数倍，主要用于访问内存映射 I/O 或访问小尺寸 Buffer 的场景中。为了解决地址非对齐问题，主设备必须在第一次传输中于 `strb` 字段里指定对应的字节有效，而后续的传输可以按照正常的传输进行。主设备给出的地址可以是非对齐的，但是其低位地址线上的信息必须与 `strb` 字段的信息保持一致。注意，AXI4 协议不要求 Subordinate 根据来自 Manager 的任何对齐信息采取特殊操作。



### 参考文献

[1] [AMBA Specifications – Arm®](https://www.arm.com/architecture/system-architectures/amba/amba-specifications)

[2] [Vivado Design Suite: AXI Reference Guide (UG1037)](https://china.xilinx.com/support/documents/ip_documentation/axi_ref_guide/latest/ug1037-vivado-axi-reference-guide.pdf)

[3] [AXI Interconnect v2.1 LogiCORE IP Product Guide](https://www.xilinx.com/support/documents/ip_documentation/axi_interconnect/v2_1/pg059-axi-interconnect.pdf)

[4] [SmartConnect v1.0 LogiCORE IP Product Guide](https://www.xilinx.com/support/documents/ip_documentation/smartconnect/v1_0/pg247-smartconnect.pdf)
