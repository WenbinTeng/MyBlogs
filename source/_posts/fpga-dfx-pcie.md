---
title: fpga-dfx-pcie
date: 2025-12-31 09:13:51
tags:
typora-root-url: ./fpga-dfx-pcie
---

# 使用 PCIe 进行部分动态重构

数据中心 FPGA 中多数配置了 PCIe IP 用于与主机进行数据传输，还可以利用 PCIe 链路用于 FPGA 配置。本博客将介绍在 Xilinx FPGA 上使用 PCIe 进行 DFX 配置的流程。

UltraScale 器件引入了 **Media Configuration Access Port (MCAP)**，它是从器件上的某一特定 PCIe 块到配置引擎的专用连接，能够提供一种有效的机制用于交付部分比特流。将 PCIe 块连接到配置引擎时无需显式布线，这样即可节省大量资源。



### 1. PCIe 配置子系统设计

PCIe 属于即插即用协议，即上电时， PCIe 主机将枚举系统。在此流程中，主机将从每个器件读取请求的地址大小，然后向器件分配基址。因此，当主机查询 PCIe 接口时，这些接口必须处于就绪状态，否则不会为其分配基址。PCI Express 规范声明，当系统电源正常的 100 ms 后，PERST# 必须断言无效，并且在 PERST# 断言无效后的 20 ms 内 PCIe 端口必须准备好进行链路训练。这通常被称为 100 ms 启动时间要求。

因此，Xilinx 使用基于**串联配置**（Tandem Configuration）的 2 阶段比特流交付方法，以满足 PCIe 规范中所述的配置时间需求。串联配置支持以下几种模式：

- **Tandem PROM**：从 PROM 中加载两个阶段的比特流。
- **Tandem PCIe**：从 PROM 中加载第一阶段比特流，然后通过 PCIe 链路将第二阶段比特流交付至 MCAP。
- **Tandem PCIe with Field Updates**：在完成 Tandem PROM/PCIe 初始配置后，在 PCIe 链路保持有效时更新整个用户设计。更新区域（布局规划）和设计结构均已预定义，并且已提供 Tcl 脚本。
- **DFX over PCIe**：用于为 DFX 启用 MCAP 链路，而无需启用串联配置。

这里我们选择 **DFX over PCIe** 模式，具体流程如下所示。

首先，在 Vivado 中创建一个 Block Design，并添加所需的 IP。通过 IP Catalog 搜索 XDMA，并选择如图所示的 XDMA IP 核。

![1](1.png)

随后，双击 XDMA IP 进入配置界面，将 Mode 设置为 Advanced，并将串联配置设置为 DFX over PCIe，配置过程如下图所示。

![2](2.png)

![3](3.png)

完成 XDMA IP 的参数配置后，运行 run block automation，由 Vivado 自动生成与目标板卡相关的连接关系与约束。生成结果如图所示。

![4](4.png)

为了验证基于 PCIe 的 DFX 配置流程，我们搭建了一个简化的测试平台，其系统框图如下图所示。

在该设计中，PCIe IP 核主要承担两类功能：

1. **DMA 数据通路**：通过 PCIe 将数据传输至用户逻辑，并将计算结果回传至主机；
2. **DFX 配置通路**：利用 PCIe 链路传输部分比特流，并触发 FPGA 的部分动态重构。

其中，`hier_0` 为用户侧的 Block Design，被配置为 DFX 的可重构模块（Reconfigurable Module, RM）。该模块内部集成了一个 `vadd` HLS 核，用于实现 `c = a + b` 的功能，并通过 `s_axilite` 接口完成寄存器控制与参数配置。

此外，我们还构建了另一个可重构模块，其中集成 `vsub` 核，实现 `c = a - b` 的功能。通过在 Host 端切换加载不同的部分比特流，可以验证用户逻辑区域是否被成功替换。DFX 相关的完整设计流程可参考 Xilinx 官方文档 UG909。

![5](5.png)

完成 Block Design 的创建后，通过 **DFX Wizard** 配置 DFX 运行配置。配置完成后的结果如下图所示。

![6](6.png)

随后点击 **Generate Bitstream**，执行综合、实现与比特流生成流程。生成完成后：

- 在路径 `<project>/<project>.runs/impl_1` 下，可以找到完整比特流以及与 `vadd` 对应的部分比特流；
- 在路径 `<project>/<project>.runs/child_0_impl_1` 下，可以找到与 `vsub` 对应的部分比特流。



### 2. 安装 XVSEC 驱动

Xilinx Vendor Specific Capabilities（XVSEC）是 Xilinx 在 PCI Express 配置空间中引入的一组扩展能力，涵盖 MCAP、ZERO VSEC 等功能。通过 XVSEC，Host 端可以直接对 FPGA 进行配置操作。

我们从 Xilinx 官方仓库中下载 XVSEC：[dma_ip_drivers](https://github.com/Xilinx/dma_ip_drivers.git)。

```bash
git clone https://github.com/Xilinx/dma_ip_drivers.git
cd dma_ip_drivers
```

进入 XVSEC 驱动目录：

```bash
cd XVSEC/linux-kernel/
```

编译驱动代码：

```bash
make clean all
```

安装驱动：

```bash
sudo make install
```

加载驱动：

```bash
sudo modprobe xvsec
```

至此，XVSEC 驱动已成功安装并加载。



### 3. 使用 MCAP 进行 FPGA 配置

首先，我们验证初始状态下 `vadd` 模块的功能。在 Vivado 中烧写完整比特流后，在 Host 端运行如下测试程序：

```c++
#include <iostream>

#include "FpgaConfig.hpp"
#include "XHlsIpConfig.hpp"

int main(int argc, char const *argv[])
{
    XHlsIp hier_inst{0x0000'0000, {}};
    const uint64_t A_CTRL_ADDR = 0x0000'0010;
    const uint64_t B_CTRL_ADDR = 0x0000'001C;
    const uint64_t C_CTRL_ADDR = 0x0000'0028;

    int a = 2;
    int b = 1;
    int c = 0;

    FpgaConfig::writeFpga(&a, sizeof(int), A_CTRL_ADDR);
    FpgaConfig::writeFpga(&b, sizeof(int), B_CTRL_ADDR);
    FpgaConfig::writeFpga(&c, sizeof(int), C_CTRL_ADDR);
    XHlsIpConfig::start(&hier_inst);
    while (!XHlsIpConfig::isDone(&hier_inst)) {
        usleep(1000);
    }

    std::cout << "c result: " << c << std::endl;
    return 0;
}
```

程序返回结果为 `c = a + b`，表明当前加载的用户逻辑为 `vadd` 模块。（在此之前，可能需要提前安装并加载 XDMA 驱动，相关步骤本文不再展开。）

接下来，使用 XVSEC 工具加载 `vsub` 对应的部分比特流，并验证动态重构是否生效。

首先，通过 `lspci` 命令查看系统中识别到的 FPGA PCIe 设备：

```bash
lspci | grep -i xilinx
```

可观察到类似如下的输出：

```bash
08:00.0 Serial controller: Xilinx Corporation Device 903f
```

其中，总线号为 `bus_no = 0x08`，设备号为 `dev_no = 0x0`。

随后，使用以下命令列出该 PCIe 设备所支持的 VSEC 能力：

```bash
sudo xvsecctl -b 0x08 -F 0x0 -l
```

输出结果示例如下：

```bash
No of Supported Extended capabilities : 3
VSEC ID   VSEC Rev   VSEC Name              Driver Support
--------  --------   ------------------    --------------
0x0001    0x0001     PCIe_MCAP_VSEC            Yes
0x0000    0x0000     UNKNOWN                   No
0x0008    0x0000     PCIe_XVC_DEBUG_VSEC       No
```

可以看到，该设备支持 `PCIe_MCAP_VSEC`，且驱动已正确识别。

最后，通过 XVSEC 的 `-p` 选项加载 `vsub` 对应的部分比特流：

```bash
sudo xvsecctl -b 0x08 -F 0x0 -p vsub_partial.bit
```

加载完成后，再次运行前述测试程序，可以观察到返回结果变为 `c = a - b`，表明可重构模块已被成功替换，PCIe over DFX 的配置流程验证完成。

