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

AXI4 与 AXI-Lite 总线协议都包含了**五个**独立的通道，每条通道都能用于单独握手与数据流动，分别负责不同的功能：

- 读地址通道（Read Address Channel）：用于传输读事务的地址与控制信息。
- 读数据通道（Read Data Channel）：用于返回读取的数据以及读响应。
- 写地址通道（Write Address Channel）：用于传输写事务的地址与控制信息。
- 写数据通道（Write Data Channel）：用于传输实际要写入的数据。
- 写响应通道（Write Response Channel）：用于反馈写操作的结果。

这些主从通道是双向的，而且每次事务中数据传输的大小是可变的。AXI4 协议可以发起突发传输，传输至多 256 个数据；AXI4-Lite 只允许每个事务传输一个数据。

读通道如下图所示：

<img src="write_channel.png" style="zoom:60%;" />

写通道如下图所示：

<img src="read_channel.png" style="zoom:60%;" />

这种通道分离设计的优势是能够提升并行性：地址传输、数据传输与响应传输相互独立，不会互相阻塞，能够提高吞吐率和系统利用率。



### 2. AXI4 总线的握手机制

AXI4 通信的每条通道都基于统一的 **VALID-READY** 握手机制：valid 信号由发送方拉高，表示数据/地址/响应已经准备好，可以被接受；ready 信号由接收方拉高，表示准备好接收数据/地址/响应。只有当 valid 信号与 ready 信号同时为高时，传输的数据才会在相应的时钟周期正式被采样（注意，在主机和从机的视角下都握手成功，传输的数据才有效）。

值得注意的是，握手信号 valid 和 ready 之间是存在依赖关系的：

- 一个组件的 valid 信号产生不能依赖于另一个组件的 ready 信号；
- ready 信号可以等待 valid 信号的断言。

如果两个组件之间的 valid 信号相互等待彼此的 ready 信号，则会造成死锁。



### 3. AXI4 事务流程



### 4. AXI4 时序设计规则
