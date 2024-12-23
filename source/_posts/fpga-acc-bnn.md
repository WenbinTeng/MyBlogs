---
title: fpga-acc-bnn
date: 2024-12-23 18:54:17
tags:
---

# 二值神经网络加速

二值神经网络（Binary Neural Network, BNN）是一种旨在大幅降低神经网络计算复杂度和存储需求的技术，特别适合在资源受限的硬件（如嵌入式设备或FPGA）上运行。以下是 BNN 的基本概念和其加速方法的介绍。



### 1. 二值神经网络的概念

二值神经网络是一种特殊的神经网络，它通过将网络中的权重和激活值限制为二进制值（+1 或 -1，通常表示为 0 和 1），显著降低了计算和存储的复杂度。与传统神经网络中的浮点数权重和激活值不同，BNN 仅使用二进制操作（如异或和加法），从而大幅减少了计算量。



### 2. FPGA 加速 BNN 架构

下面我们对 rosetta 基准测试 [1] 进行修改，在 FPGA 上加速 BNN 架构。





### 3. 参考文献

[1] Zhou Y, Gupta U, Dai S, et al. Rosetta: A realistic high-level synthesis benchmark suite for software programmable FPGAs[C]//Proceedings of the 2018 ACM/SIGDA International Symposium on Field-Programmable Gate Arrays. 2018: 269-278.
