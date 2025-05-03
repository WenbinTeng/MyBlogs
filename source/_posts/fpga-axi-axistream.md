---
title: fpga-axi-axistream
date: 2025-05-03 14:48:23
tags:
---

# AXI 总线学习 (6) - AXI4-Stream

AXI4-Stream 是从 AMBA 总线协议家族中专为点对点、高吞吐量数据流设计的接口标准。与面向存储器映射的 AXI4 协议不同，AXI4-Stream 不包含地址信息，仅关注数据的握手与传输，因而更轻量、延迟更低。



### 1. AXI4-Stream 协议概览

AXI4-Stream 规定了两条单向信号通道：

- **TVALID/TREADY 握手通道**：保证数据在双方都准备好的时刻传输；
- **TDATA 及可选用户信号通道**：承载实际数据及附加控制信息。

数据传输过程中，主设备在输出数据有效时将 TVALID 拉高，从设备在能够接收时将 TREADY 拉高。当两者同时为高电平时，完成一次数据打拍。

##### 握手通道

| 信号   | 宽度 | 源          | 描述                         |
| ------ | ---- | ----------- | ---------------------------- |
| TVALID | 1    | Transmitter | 主设备地址有效               |
| TREADY | 1    | Receiver    | 从设备准备好接收地址（可选） |

##### 数据通道

| 信号  | 宽度            | 源          | 描述                                 |
| ----- | --------------- | ----------- | ------------------------------------ |
| TDATA | TDATA_WIDTH     | Transmitter | 读数据                               |
| TSTRB | TDATA_WIDTH / 8 | Transmitter | 字节选通信号（每位对应一个字节）     |
| TKEEP | TDATA_WIDTH / 8 | Transmitter | 字节传送控制信号（每位对应一个字节） |
| TLAST | 1               | Transmitter | 最后一个数据标志                     |
| TID   | ID_T_WIDTH      | Transmitter | 数据流标识 ID                        |
| TDEST | TDEST_WIDTH     | Transmitter | 提供数据流的路由信息                 |
| TUSER | TUSER_WIDTH     | Transmitter | 用户定义信号                         |



### 参考文献

[1] [AMBA Specifications – Arm®](https://www.arm.com/architecture/system-architectures/amba/amba-specifications)

[2] [Vivado Design Suite: AXI Reference Guide (UG1037)](https://china.xilinx.com/support/documents/ip_documentation/axi_ref_guide/latest/ug1037-vivado-axi-reference-guide.pdf)

[3] [AXI Interconnect v2.1 LogiCORE IP Product Guide](https://www.xilinx.com/support/documents/ip_documentation/axi_interconnect/v2_1/pg059-axi-interconnect.pdf)

[4] [SmartConnect v1.0 LogiCORE IP Product Guide](https://www.xilinx.com/support/documents/ip_documentation/smartconnect/v1_0/pg247-smartconnect.pdf)
