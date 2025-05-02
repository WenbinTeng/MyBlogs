---
title: fpga-axi-intro
date: 2025-04-28 10:13:11
tags:
---

# AXI 总线学习 (1) - 总览

在当今的系统级芯片（SoC）设计中，模块之间的高效通信是系统性能的关键。为了满足高带宽、低延迟以及模块间复杂通信的需求，ARM 公司提出了AMBA（Advanced Microcontroller Bus Architecture）总线协议家族，其中AXI（Advanced eXtensible Interface）作为其第三代标准（AMBA 3.0），已经事实上成为 AMBA 总线中最常见、使用最频繁的片上互联协议。特别是 AXI4 版本，凭借其灵活性和高效性，广泛应用于 FPGA、ASIC 等领域的设计中。

从 Spartan-6 和 Virtex-6 设备开始，Xilinx 就在后续生产的 SoC 设备上使用 AXI 协议。



### AXI4 架构概览

AXI4 通过将一次读写事务拆分为五条独立通道来实现并行：

- **读地址通道**（Read Address Channel）
- **读数据通道**（Read Data Channel）
- **写地址通道**（Write Address Channel）
- **写数据通道**（Write Data Channel）
- **写响应通道**（Write Response Channel）

五条通道互不阻塞，可同时进行握手与数据传输，可以有效降低等待时间。



### AXI4 核心特性

参考 AXI4 文档 [1]，AXI4 总线协议具有以下特点：

- 独立的地址/控制和数据通道；
- 使用字节选通来支持非对齐数据传输；
- 可以使用一个地址发起突发数据传输；
- 可以提供低成本直接内存访问（DMA）的独立的读写数据通道；
- 可以发起多个超前传输；
- 支持事务乱序完成；
- 可以方便地添加流水段寄存器以完成时序收敛。



### AXI4 协议家族

AMBA AXI4 协议其实是一个家族，主要包括以下三个子协议：

- **AXI4**：用于高性能、高带宽的内存映射型数据传输，支持突发传输，适合大数据量、复杂传输模式场景。

- **AXI4-Lite**：简化版AXI4协议，仅支持简单的单次读写访问，通常用于低带宽、低复杂度的控制信号传输，如配置寄存器访问。

- **AXI4-Stream**：专为点对点的高速数据流设计，去除了地址通道，专注于数据的连续传输，广泛应用于数据流处理场景，比如视频处理、网络数据传输等。



在接下来的博客中，将分逐步介绍 AXI 总线的基本架构、信号定义以及进阶特性，帮助大家系统掌握这一关键通信协议。



### 参考文献

[1] [AMBA Specifications – Arm®](https://www.arm.com/architecture/system-architectures/amba/amba-specifications)
