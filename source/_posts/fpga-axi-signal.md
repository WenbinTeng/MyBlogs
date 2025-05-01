---
title: fpga-axi-signal
date: 2025-04-30 16:27:45
tags:
---

# AXI 总线学习 (3) - 信号定义

本文聚焦于 AXI4 协议中各个通道所使用的具体信号，介绍每个信号的作用、方向、以及在数据传输中的意义。本文主要以 Xilinx 中 AXI Interconnect IP 中实现的信号作为例子，更多的信号定义请参考 ARBA 总线设计文档。



### 1. 全局信号

全局信号包含时钟信号与全局复位信号。

| 信号    | 宽度 | 源   | 描述                       |
| ------- | ---- | ---- | -------------------------- |
| ACLK    | 1    | 外部 | 全局时钟信号。             |
| ARESETn | 1    | 外部 | 全局复位信号，低电平有效。 |



### 2. 读地址通道信号

读地址通道用于传输必要的地址信息与控制信息，该通道上所有信号的前缀为“AR”。

| 信号    | 宽度           | 源          | 描述                 |
| ------- | -------------- | ----------- | -------------------- |
| ARVALID | 1              | Manager     | 主设备地址有效       |
| ARREADY | 1              | Subordinate | 从设备准备好接收地址 |
| ARADDR  | ADDR_WIDTH     | Manager     | 读地址               |
| ARPROT  | 3              | Manager     | 保护类型             |
| ARLEN   | 8              | Manager     | 突发长度             |
| ARSIZE  | 3              | Manager     | 数据大小             |
| ARBURST | 2              | Manager     | 突发类型             |
| ARLOCK  | 1              | Manager     | 锁标志               |
| ARCACHE | 4              | Manager     | 缓存类型             |
| ARQOS   | 4              | Manager     | QoS 信息             |
| ARID    | ID_R_WIDTH     | Manager     | 事务 ID              |
| ARUSER  | USER_REQ_WIDTH | Manager     | 用户定义信号         |



### 3. 读数据通道信号

读数据通道用于从设备向主设备返回读取结果，该通道上所有信号的前缀为“R”。

| 信号   | 宽度                              | 源          | 描述                 |
| ------ | --------------------------------- | ----------- | -------------------- |
| RVALID | 1                                 | Subordinate | 从设备数据有效       |
| RREADY | 1                                 | Manager     | 主设备准备接收数据   |
| RDATA  | DATA_WIDTH                        | Subordinate | 读数据               |
| RRESP  | RRESP_WIDTH                       | Subordinate | 响应码               |
| RLAST  | 1                                 | Subordinate | 最后一个突发数据标志 |
| RID    | ID_R_WIDTH                        | Subordinate | 事务 ID              |
| RUSER  | USER_DATA_WIDTH + USER_RESP_WIDTH | Subordinate | 用户定义信号         |



### 4. 写地址通道信号

写地址通道负责主设备向从设备发送写事务的地址信息，该通道上所有信号的前缀为“AW”。

| 信号    | 宽度           | 源          | 描述                           |
| ------- | -------------- | ----------- | ------------------------------ |
| AWVALID | 1              | Manager     | 主设备发出地址有效信号         |
| AWREADY | 1              | Subordinate | 从设备准备好接收地址           |
| AWADDR  | ADDR_WIDTH     | Manager     | 写地址                         |
| AWPROT  | 3              | Manager     | 保护类型（例如权限控制）       |
| AWLEN   | 8              | Manager     | 突发传输的长度                 |
| AWSIZE  | 3              | Manager     | 每次传输的数据大小（字节数）   |
| AWBURST | 2              | Manager     | 突发传输类型（固定/递增/环形） |
| AWLOCK  | 1              | Manager     | 原子操作锁标志                 |
| AWCACHE | 4              | Manager     | 缓存类型标志                   |
| AWQOS   | 4              | Manager     | QoS 信息                       |
| AWID    | ID_W_WIDTH     | Manager     | 事务 ID，用于多路复用支持      |
| AWUSER  | USER_REQ_WIDTH | Manager     | 用户定义信号                   |



### 5. 写数据通道信号

该通道传输实际写入的数据，该通道上所有信号的前缀为“W”。

| 信号   | 宽度            | 源          | 描述                             |
| ------ | --------------- | ----------- | -------------------------------- |
| WVALID | 1               | Manager     | 主设备数据有效                   |
| WREADY | 1               | Subordinate | 从设备准备好接收数据             |
| WDATA  | DATA_WIDTH      | Manager     | 写数据                           |
| WSTRB  | DATA_WIDTH / 8  | Manager     | 写字节使能位（每位对应一个字节） |
| WLAST  | 1               | Manager     | 写突发中的最后一个数据标志       |
| WID    | ID_W_WIDTH      | Manager     | 与写地址通道对应的 ID（可选）    |
| WUSER  | USER_DATA_WIDTH | Manager     | 用户定义信号                     |



### 6. 写响应通道信号

该通道用于传输从设备对写事务的响应，该通道上所有信号的前缀为“B”。

| 信号   | 宽度            | 源          | 描述                            |
| ------ | --------------- | ----------- | ------------------------------- |
| BVALID | 1               | Subordinate | 从设备响应有效                  |
| BREADY | 1               | Manager     | 主设备准备好接收响应            |
| BRESP  | BRESP_WIDTH     | Subordinate | 响应码（OKAY/SLVERR/DECERR 等） |
| BID    | ID_W_WIDTH      | Subordinate | 与写请求对应的事务 ID           |
| BUSER  | USER_RESP_WIDTH | Subordinate | 用户定义信号                    |



### 参考文献

[1] [AMBA Specifications – Arm®](https://www.arm.com/architecture/system-architectures/amba/amba-specifications)

[2] [Vivado Design Suite: AXI Reference Guide (UG1037)](https://china.xilinx.com/support/documents/ip_documentation/axi_ref_guide/latest/ug1037-vivado-axi-reference-guide.pdf)

[3] [AXI Interconnect v2.1 LogiCORE IP Product Guide](https://www.xilinx.com/support/documents/ip_documentation/axi_interconnect/v2_1/pg059-axi-interconnect.pdf)

[4] [SmartConnect v1.0 LogiCORE IP Product Guide](https://www.xilinx.com/support/documents/ip_documentation/smartconnect/v1_0/pg247-smartconnect.pdf)
