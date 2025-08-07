---
title: fpga-gate-count
date: 2025-08-07 22:09:25
tags:
---

# FPGA 逻辑资源的等效门数量

Xilinx FPGA 手册中一般是以 Site-Level Logic （SLICEL，SLICEM，DSP SLICE，BRAM）或者 Tile-Level Logic （CLB，DSP，BRAM Blocks）作为资源使用统计的基本单位。那么如何将这些基本单元换算为等效门数量呢？

从 DS180 中我们可以得知，每个 7 系 FPGA slice 包含 4 个 LUT 与 8 个 flip-flops；并且，我们发现 logic cell 数量与 slices 数量的对应关系是 6.4 倍。

从 XAPP059 中我们可以得知，一个 logic cell 的等效门数量典型值为 12 个。

因此我们可以得出从 slice 计算等效门数量的公式：

$$
    1 \times slice = 6.4 \times logic\ cells = 76.8 \times gates
$$

### 参考文献

[1] [7 Series FPGAs Data Sheet: Overview (DS180)](https://docs.amd.com/v/u/en-US/ds180_7Series_Overview)
[2] [Gate Count Capacity Metrics for FPGAs Application Note (XAPP059)](https://docs.amd.com/v/u/en-US/xapp059)
