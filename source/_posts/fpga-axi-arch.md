---
title: fpga-axi-arch
date: 2025-04-29 08:20:06
tags:
typora-root-url: fpga-axi-arch
---

# AXI 总线学习 (2) - 基本架构

本文将会介绍 AXI4 总线的基本架构，包括通道、握手机制和事务等。



### 1. AXI4 总线的读写通道

AXI4 协议规范描述了单个 AXI4 主机与从机之间的接口，这里的主机和从机表示的是需要交换信息的 IP 核心。如果你使用的是 Xilinx 的 FPGA 开发环境，它的环境中提供了 AXI 互联的 IP，以供使用内存映射的多个主机和多个从机相互连接。这些互联 IP 的接口数量是可以配置的，可以将多个主机接口和多个从机接口之间的事务进行路由。详细可以参考 Xilinx AXI Interconnect IP 以及 Xilinx AXI SmartConnect IP。

（注意，从 2021 年开始，ARBA 总线的“主从”名称由“Master-Slave”改为“Manager-Subordinate” [1]。）

AXI4 与 AXI-Lite 总线协议都包含了**五个**独立的通道，每条通道都能用于单独握手与数据流动，分别负责不同的功能：

- 读地址通道（Read Address Channel）：用于传输读事务的地址与控制信息，相关信号以 **AR** 开头。
- 读数据通道（Read Data Channel）：用于返回读取的数据以及读响应，相关信号以 **R** 开头。
- 写地址通道（Write Address Channel）：用于传输写事务的地址与控制信息，相关信号以 **AW** 开头。
- 写数据通道（Write Data Channel）：用于传输实际要写入的数据，相关信号以 **W** 开头。
- 写响应通道（Write Response Channel）：用于反馈写操作的结果，相关信号以 **B** 开头。

这些主从通道是双向的，而且每次事务中数据传输的大小是可变的。AXI4 协议可以发起突发传输，传输至多 256 个周期的数据 [2]；AXI4-Lite 只允许每个事务传输一个数据。通过配置相应的 DATA_WIDTH 字段，可以选择至多 1024 bits 的数据传输位宽 [2]。

读通道如下图所示：

<img src="read_channel.png" style="zoom:60%;" />

写通道如下图所示：

<img src="write_channel.png" style="zoom:60%;" />

这种通道分离设计的优势是能够提升并行性：地址传输、数据传输与响应传输相互独立，不会互相阻塞，能够提高吞吐率和系统利用率。

在一个系统中，主从设备可以通过一个互联结构（Interconnect）相互连接，Interconnect 拥有与主从设备对称的 AXI 接口以供连接。Interconnect 内部的连接取决于具体的实现，而 AXI 协议只规定相应的接口和传输方式，如下图所示：

<img src="interconnect.png" style="zoom:60%;" />

一个比较经典的 Interconnect 实现是 Xilinx 的 AXI Interconnect / Smartconnect IP [3-4]，其中 AXI Interconnect 的顶层结构如图所示。在其内部，一个 Crossbar 核心在从接口（SI）和主接口（MI）之间路由流量。沿着连接 SI 或 MI 到 Crossbar 的每个路径，可选的 AXI 基础设施核心（耦合器）系列可以执行各种转换和缓冲功能。可能包含的耦合器有：寄存器切片、数据 FIFO、时钟转换器、数据宽度转换器和协议转换器。AXI Interconnect 核心可以配置为最多16个 SI 和最多16个 MI。

<img src="xilinx_interconnect_top.png" style="zoom:80%;" />

每个 AXI Interconnect 核心可以根据需求配置为以下几种工作模式：

- N-to-1 Interconnect
- 1-to-N Interconnect
- N-to-M Interconnect (Crossbar Mode)
- N-to-M Interconnect (Shared Access Mode)

它们的基本结构如下图所示：

<img src="2-2.png" style="zoom:50%;" />



<img src="2-3.png" style="zoom:50%;" />



<img src="2-4.png" style="zoom:50%;" />



<img src="2-5.png" style="zoom:50%;" />



### 2. AXI4 总线的握手机制

AXI4 通信的每条通道都基于统一的 **VALID-READY** 握手机制：valid 信号由发送方拉高，表示数据/地址/响应已经准备好，可以被接受；ready 信号由接收方拉高，表示准备好接收数据/地址/响应。只有当 valid 信号与 ready 信号同时为高时，传输的数据才会在相应的时钟周期正式被采样（注意，在主机和从机的视角下都握手成功，传输的数据才有效）。

值得注意的是，握手信号 valid 和 ready 之间是存在依赖关系的：

- 一个组件的 valid 信号产生不能依赖于另一个组件的 ready 信号；
- ready 信号可以等待 valid 信号的断言。

如果两个组件之间的 valid 信号相互等待彼此的 ready 信号，则会造成死锁。

下图给出了读写事务中 valid 和 ready 信号之间的依赖关系，其中，单箭头表示汇节点可以在源节点之前或之后被断言，双箭头表示汇节点只能在源节点被断言之后才可被断言。

读事务信号的依赖关系如下图所示：

<img src="read_dependency.png" style="zoom:50%;" />

写事务信号的依赖关系如下图所示：

<img src="write_dependency.png" style="zoom:50%;" />



### 参考文献

[1] [AMBA Specifications – Arm®](https://www.arm.com/architecture/system-architectures/amba/amba-specifications)

[2] [Vivado Design Suite: AXI Reference Guide (UG1037)](https://china.xilinx.com/support/documents/ip_documentation/axi_ref_guide/latest/ug1037-vivado-axi-reference-guide.pdf)

[3] [AXI Interconnect v2.1 LogiCORE IP Product Guide](https://www.xilinx.com/support/documents/ip_documentation/axi_interconnect/v2_1/pg059-axi-interconnect.pdf)

[4] [SmartConnect v1.0 LogiCORE IP Product Guide](https://www.xilinx.com/support/documents/ip_documentation/smartconnect/v1_0/pg247-smartconnect.pdf)
