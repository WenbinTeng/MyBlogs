---
title: fpga-onchip-memory
date: 2025-04-27 16:04:02
tags:
typora-root-url: ./fpga-onchip-memory
---

# FPGA 片上存储资源 FPGA On-chip Memory

本文将介绍 FPGA 片上的常见存储资源。



### 1. D 触发器

在 Xilinx 的 7 系 FPGA 上拥有大量的多种的 D 触发器，用以支持不同功能的时序逻辑。

##### 1.1 FDCE

具有时钟使能和**异步复位**的 D 触发器（D Flip-Flop with Clock Enable and Asynchronous Clear，FDCE），当时钟使能信号（CE）为高且异步复位信号（CLR）未生效时，触发器的数据输入（D）在时钟信号（C）上升沿会被传输到相应的数据输出（Q）。当 CLR 为高电平时，它会覆盖其他所有输入，将 Q 置为低电平。当 CE 为低电平时，会忽略 C 的变换。其逻辑框图和逻辑功能表如下：

<img src="1.png" style="zoom: 70%;" />

<style type="text/css">
.tg  {border-collapse:collapse;border-spacing:0;}
.tg td{border-style:solid;border-width:0px;font-family:Arial, sans-serif;font-size:14px;overflow:hidden;
  padding:10px 5px;word-break:normal;}
.tg th{border-style:solid;border-width:0px;font-family:Arial, sans-serif;font-size:14px;font-weight:normal;
  overflow:hidden;padding:10px 5px;word-break:normal;}
.tg .tg-c3ow{border-color:inherit;text-align:center;vertical-align:top}
</style>
<table class="tg"><thead>
  <tr>
    <th class="tg-c3ow" colspan="4">Inputs</th>
    <th class="tg-c3ow">Outputs</th>
  </tr></thead>
<tbody>
  <tr>
    <td class="tg-c3ow">R</td>
    <td class="tg-c3ow">CE</td>
    <td class="tg-c3ow">D</td>
    <td class="tg-c3ow">C</td>
    <td class="tg-c3ow">Q</td>
  </tr>
  <tr>
    <td class="tg-c3ow">1</td>
    <td class="tg-c3ow">X</td>
    <td class="tg-c3ow">X</td>
    <td class="tg-c3ow">X</td>
    <td class="tg-c3ow">0</td>
  </tr>
  <tr>
    <td class="tg-c3ow">0</td>
    <td class="tg-c3ow">0</td>
    <td class="tg-c3ow">X</td>
    <td class="tg-c3ow">X</td>
    <td class="tg-c3ow">No Change</td>
  </tr>
  <tr>
    <td class="tg-c3ow">0</td>
    <td class="tg-c3ow">1</td>
    <td class="tg-c3ow">D</td>
    <td class="tg-c3ow">&#8593;</td>
    <td class="tg-c3ow">D</td>
  </tr>
</tbody>
</table>

在 Vivado 中，以下 always 块会被综合为 FDCE：

```verilog
always @(posedge clk or posedge rst) begin
    if (rst)
        Q <= 0;
    else
        Q <= D;
end
```

##### 1.2 FDRE

具有时钟使能和**同步复位**的 D 触发器（D Flip-Flop with Clock Enable and Synchronous Reset，FDRE），当时钟使能信号（CE）为高且同步复位信号（R）未生效时，触发器的数据输入（D）在时钟信号（C）上升沿会被传输到相应的数据输出（Q）。当 R 在时钟上升沿到来时为高电平，它会覆盖其他所有输入，将 Q 置为低电平。当 CE 为低电平时，会忽略 C 的变换。其逻辑框图和逻辑功能表如下：

<img src="2.png" style="zoom: 70%;" />

<style type="text/css">
.tg  {border-collapse:collapse;border-spacing:0;}
.tg td{border-style:solid;border-width:0px;font-family:Arial, sans-serif;font-size:14px;overflow:hidden;
  padding:10px 5px;word-break:normal;}
.tg th{border-style:solid;border-width:0px;font-family:Arial, sans-serif;font-size:14px;font-weight:normal;
  overflow:hidden;padding:10px 5px;word-break:normal;}
.tg .tg-c3ow{border-color:inherit;text-align:center;vertical-align:top}
</style>
<table class="tg"><thead>
  <tr>
    <th class="tg-c3ow" colspan="4">Inputs</th>
    <th class="tg-c3ow">Outputs</th>
  </tr></thead>
<tbody>
  <tr>
    <td class="tg-c3ow">R</td>
    <td class="tg-c3ow">CE</td>
    <td class="tg-c3ow">D</td>
    <td class="tg-c3ow">C</td>
    <td class="tg-c3ow">Q</td>
  </tr>
  <tr>
    <td class="tg-c3ow">1</td>
    <td class="tg-c3ow">X</td>
    <td class="tg-c3ow">X</td>
    <td class="tg-c3ow">&#8593;</td>
    <td class="tg-c3ow">0</td>
  </tr>
  <tr>
    <td class="tg-c3ow">0</td>
    <td class="tg-c3ow">0</td>
    <td class="tg-c3ow">X</td>
    <td class="tg-c3ow">X</td>
    <td class="tg-c3ow">No Change</td>
  </tr>
  <tr>
    <td class="tg-c3ow">0</td>
    <td class="tg-c3ow">1</td>
    <td class="tg-c3ow">D</td>
    <td class="tg-c3ow">&#8593;</td>
    <td class="tg-c3ow">D</td>
  </tr>
</tbody>
</table>

在 Vivado 中，以下 always 块会被综合为 FDRE：

```verilog
always @(posedge clk) begin
    if (rst)
        Q <= 0;
    else
        Q <= D;
end
```

##### 1.3 FDPE

具有时钟使能和**异步置位**的 D 触发器（D Flip-Flop with Clock Enable and Asynchronous Preset，FDPE），当置位信号 PRE 为高电平时，将输出异步置为 1。其他行为类似于 FDCE。

##### 1.4 FDSE

具有时钟使能和**同步置位**的 D 触发器（D Flip-Flop with Clock Enable and Synchronous Set，FDSE），当置位信号 S 为高电平时，将输出同步置为 1。其他行为类似于 FDRE。



### 2. 寄存器堆

在数字系统中，通常把能够用来存储二进制数据的同步时序逻辑电路称为寄存器，是一种存储器。在 FPGA 上，一般使用 **D 触发器**来实现寄存器。一个寄存器只能存储一位数据，如果需要存储多个的多位的数据，则需要使用多个寄存器。拥有统一读写接口的多个寄存器组合可以被称为**寄存器堆**。

与其他常见存储器一样，寄存器堆也需要通过“地址”来访问相应的“数据”。寄存器堆的逻辑框图及其内部组成如下。其中，AW 是地址位宽，DW 是数据位宽。

在写数据端，寄存器堆通过一个类似解码器的组合电路，将传入的地址解码为独热码，以选中相应的寄存器，如果写使能有效，则将传入的数据写入被选中的寄存器。在读数据端，寄存器堆通过一个类似多路复用器的组合电路，根据传入的地址选择相应的寄存器数据作为输出。

<img src="3.png" style="zoom: 100%;" />

在 Verilog 中，我们可以使用向量型数组来表示一个寄存器堆。假设寄存器堆由时钟上升沿触发，采用异步的复位方式，寄存器位宽 $WIDTH=2$，寄存器深度 $DEPTH=2$，其 Verilog 代码如下：

```verilog
module regfile (
    input           clk,
    input           rst,
    input   [0:0]   address,        // [ceil(log(DEPTH))-1:0]
    input           write_enable,
    input   [1:0]   write_data,     // [WIDTH-1:0]
    output  [1:0]   read_data       // [WIDTH-1:0]
);

    integer i;
    parameter WIDTH = 2;
    parameter DEPTH = 2;
    reg [WIDTH-1:0] reg_array [DEPTH-1:0];

    always @(posedge clk or posedge rst) begin
        if (rst)
            for (i = 0; i < DEPTH; i = i + 1) begin
                reg_array[i] <= 'b0;
            end
        else if (write_enable)
            reg_array[address] <= write_data;
    end
    
    assign read_data = reg_array[address];

endmodule
```

使用 Vivado 综合后，得到的电路图如下。可以看到，Vivado 使用 4 个 FDCE 用来实现 Verilog 代码中定义的 2*2 寄存器堆。

<img src="17.png" style="zoom: 100%;" />



### 3. Distributed RAM

在 Xilinx 的 7 系 FPGA 上，可以使用一定数量的逻辑资源来构成一个存储器，以供快速的 bit 量级的数据访问，这种存储器称为分布式随机访问存储器（Distributed Random Access Memory，DRAM）。注意，这里的 DRAM 与我们平时所指的内存介质 DRAM 不同，后者是动态随机存取存储器（Dynamic Random Access Memory），是一种利用电容内存储电荷的存储介质，在高端 FPGA 开发板上同样存在，但这里我们只讨论 FPGA 片上的存储，请注意区分。

Xilinx 的 7 系 FPGA 一般支持以下几种 DRAM：

- 单端口 RAM（Single-port RAM）：通过单组接口读写一块存储空间。
- 简易双端口 RAM（Simple Dual-port RAM）：有 A 和 B 两组接口，其中 A 接口用来写 RAM，B 接口用来读 RAM。
- 双端口 RAM（Dual-port RAM）：有 A 和 B 两组接口，每一组接口都可以完成读和写操作。
- 单端口 ROM（Single-port ROM）：单端口的只读存储器。
- 双端口 ROM（Dual-port ROM）：双端口的只读存储器。

在 Vivado 中，可以通过向自己的工程中添加 IP 核以实例化 DRAM，步骤如下。首先，在左侧的栏目中点击“IP Catalog”。

<img src="4.png" style="zoom: 100%;" />

在弹出窗口的搜索栏中输入“ram”，选择“Distributed Memory Generator”。

<img src="5.png" style="zoom: 100%;" />

接下来进入 IP 核的配置窗口，我们选择实现一个 16*16 的单端口 RAM，其配置如图所示，其他选项保持默认。

<img src="6.png" style="zoom: 100%;" />

在源代码窗口的“IP Sources”选项中可以看到我们创建的IP核文件，其中我们可以看到 Verilog 版本的实例化代码模板，如图所示。

<img src="7.png" style="zoom: 100%;" />

<img src="8.png" style="zoom: 100%;" />



### 4. Block RAM

在 Xilinx 的 7 系 FPGA 上，还存在着一种专用的存储资源，能够提供 kbit 量级的数据访问，这种存储器被称为块状随机访问存储器（Block Random Access Memory，BRAM）。BRAM 分布在 FPGA 片上的固定区域，距离逻辑资源块的布线距离可能较长，因此常被用于数据量级比 DRAM 大、延迟要求比 DRAM 低的场景。

与 DRAM 的使用类似，在 Vivado 中，可以通过向自己的工程中添加 IP 核以实例化 BRAM，步骤如下。首先，在左侧的栏目中点击“IP Catalog”。

<img src="4.png" style="zoom: 100%;" />

在弹出窗口的搜索栏中输入“ram”，选择“Block Memory Generator”。

<img src="9.png" style="zoom: 100%;" />

接下来进入 IP 核的配置窗口，我们选择实现一个 16*16 的单端口 RAM，其配置如图所示，其他选项保持默认。

<img src="10.png" style="zoom: 100%;" />

在源代码窗口的“IP Sources”选项中可以看到我们创建的 IP 核文件，其中我们可以看到 Verilog 版本的实例化代码模板，如图所示。

<img src="11.png" style="zoom: 100%;" />

<img src="12.png" style="zoom: 100%;" />

BRAM 与 DRAM 的一大不同是，DRAM 是**异步**读端口，而 BRAM 是**同步**读端口。这意味着，只有经过时钟上升沿后才能从 BRAM 端口读取正确的数据。这样就带来一个问题，当写使能有效时，此时应该输出写入的新数据，还是原先存储器中保存的旧数据？这便是 BRAM 的操作模式拥有“写优先”和“读优先”两种模式的原因，如下图所示为创建 IP 核时选择的操作模式。

<img src="13.jpg" style="zoom: 100%;" />

在“写优先”模式下，如果在时钟上升沿到来时写使能有效，则 BRAM 的读端口会输出写入的新数据，如下图所示。

<img src="14.jpg" style="zoom: 100%;" />

在“读优先”模式下，如果在时钟上升沿到来时写使能有效，则 BRAM 的读端口会输出原先存储器中保存的旧数据，如下图所示。

<img src="15.jpg" style="zoom: 100%;" />

此外，BRAM 还有一种“保持”模式，即如果在时钟上升沿到来时写使能有效，向存储器写入数据，但保持读端口的输出不变，如下图所示。

<img src="16.png" style="zoom: 100%;" />

大家可以自行尝试不同操作模式的区别。
