---
title: fpga-axi-axilite
date: 2025-05-03 09:40:13
tags:
---

# AXI 总线学习 (5) - AXI4-Lite

AXI4-Lite 是 AXI4 协议的简化版本，主要用于处理简单、低吞吐量的内存映射通信。



### 1. AXI4-Lite 协议概览

AXI4-Lite 总线聚焦于低带宽的无突发简单事务传输，因此它相比于 AXI4 总线有很大精简，具体表现为：AXI4-Lite 仅支持单次传输、地址对齐访问、简单握手机制等。

和 AXI4 保持一致，AXI4-Lite 同样由五个通道组成，但每个通道都进行了简化，仅支持最基本的传输：

##### 读地址通道（Read Address Channel）

| 信号    | 宽度 | 源          | 描述                 |
| ------- | ---- | ----------- | -------------------- |
| ARVALID | 1    | Manager     | 主设备地址有效       |
| ARREADY | 1    | Subordinate | 从设备准备好接收地址 |
| ARADDR  | 32   | Manager     | 读地址               |
| ARPROT  | 3    | Manager     | 保护类型             |

##### 读数据通道（Read Data Channel）

| 信号   | 宽度 | 源          | 描述               |
| ------ | ---- | ----------- | ------------------ |
| RVALID | 1    | Subordinate | 从设备数据有效     |
| RREADY | 1    | Manager     | 主设备准备接收数据 |
| RDATA  | 32   | Subordinate | 读数据             |
| RRESP  | 2    | Subordinate | 响应码             |

##### 写地址通道信号（Write Address Channel）

| 信号    | 宽度 | 源          | 描述                     |
| ------- | ---- | ----------- | ------------------------ |
| AWVALID | 1    | Manager     | 主设备发出地址有效信号   |
| AWREADY | 1    | Subordinate | 从设备准备好接收地址     |
| AWADDR  | 32   | Manager     | 写地址                   |
| AWPROT  | 3    | Manager     | 保护类型（例如权限控制） |

##### 写数据通道信号（Write Data Channel）

| 信号   | 宽度 | 源          | 描述                             |
| ------ | ---- | ----------- | -------------------------------- |
| WVALID | 1    | Manager     | 主设备数据有效                   |
| WREADY | 1    | Subordinate | 从设备准备好接收数据             |
| WDATA  | 32   | Manager     | 写数据                           |
| WSTRB  | 4    | Manager     | 写字节使能位（每位对应一个字节） |

##### 写响应通道信号（Write Response Channel）

| 信号   | 宽度 | 源          | 描述                            |
| ------ | ---- | ----------- | ------------------------------- |
| BVALID | 1    | Subordinate | 从设备响应有效                  |
| BREADY | 1    | Manager     | 主设备准备好接收响应            |
| BRESP  | 2    | Subordinate | 响应码（OKAY/SLVERR/DECERR 等） |



### 2. AXI4-Lite 传输流程

AXI4-Lite 只支持单次传输，因此每次读写操作都是独立完成的，不支持 AXI4 中的突发传输和乱序响应等特性，其余的传输流程与 AXI4 类似：

##### 读传输流程

1. Manager 发出读地址（ARADDR + ARVALID）
2. Subordinate 接收地址后返回 ARREADY
3. Subordinate 准备好数据后发送 RDATA + RRESP + RVALID
4. Manager 接收数据并返回 RREADY

##### 写传输流程

1. Manager 发出写地址（AWADDR + AWVALID）
2. Subordinate 接收地址后返回 AWREADY
3. Manager 发出写数据（WDATA + WVALID）
4. Subordinate 接收数据后返回 WREADY
5. Subordinate 返回写响应（BRESP + BVALID）
6. Manager 接收响应后发送 BREADY



### 参考文献

[1] [AMBA Specifications – Arm®](https://www.arm.com/architecture/system-architectures/amba/amba-specifications)

[2] [Vivado Design Suite: AXI Reference Guide (UG1037)](https://china.xilinx.com/support/documents/ip_documentation/axi_ref_guide/latest/ug1037-vivado-axi-reference-guide.pdf)

[3] [AXI Interconnect v2.1 LogiCORE IP Product Guide](https://www.xilinx.com/support/documents/ip_documentation/axi_interconnect/v2_1/pg059-axi-interconnect.pdf)

[4] [SmartConnect v1.0 LogiCORE IP Product Guide](https://www.xilinx.com/support/documents/ip_documentation/smartconnect/v1_0/pg247-smartconnect.pdf)
