---
title: fpga-dfx-ip-project-flow
date: 2025-05-09 08:58:17
tags:
typora-root-url: fpga-dfx-ip-project-flow
---

# FPGA DFX IP Project Flow

在大型项目中，常常使用 IP 来模块化地管理设计源代码，以方便地管理系统中各个模块的功能、组织模块之间的互联。Vivado 2021.2 版本更新了基于 IP 管理的项目设计，可以让我们更方便地在模块设计（Block Design，BD）中使用 DFX 功能。基于 IP 项目的 DFX 设计包含以下特性：

- 能够创建模块设计容器（Block Design Containers，BDC），并能在项目结构中识别其层次结构。
- 将 BDC 定义为可重构分区（Reconfigurable Partitions，RP）。
- 为每个 RP 创建一组可重构模块（Reconfigurable Modules，RM）。
- 创建顶层模块与子模块综合运行、创建配置、创建实现运行、验证配置、生成完整比特流与部分比特流等，与基于 RTL 的项目设计类似。

开始实验之间，我们先从 Xilinx 官网下载官方 DFX 教程设计文件：

[下载链接](https://www.xilinx.com/member/forms/download/design-license.html?cid=03e48cb4-ba89-496d-a3d7-cbaa2302ef79&filename=ug947-vivado-partial-reconfiguration-tutorial.zip)

下载完成后，将文件解压至任意具备写权限的本地路径，然后进入解压后的 `ipi_bdc_dfx_zu ` 子目录。本实验将使用 ZCU102 作为目标开发板，所有步骤都将在该目录下完成。所有编译步骤可以使用 `run_all.tcl` 以 Tcl 脚本的形式运行，这里为了展示 DFX 的功能，将使用 GUI 中的 IP 集成器（IP Integrator）进行演示。



### 1. 在 IP 集成器中创建设计

1. 启动 Vivado IDE。在 Tcl 控制台中，导航到教程存档解压的文件夹 `ipi_bdc_dfx_zu ` 。

2. 激活第一个 Tcl 脚本已创建一个针对 ZCU102 开发板的平面设计。

   ```tcl
   source create_top_bd.tcl
   ```

   这个脚本会执行一些任务：

   - 为 ZCU102 目标开发板创建一个新项目；
   - 添加并自定义一组 IP；
   - 在 BD 中连接各个 IP；
   - 验证并保存 BD 设计。

   这个 ` create_top_bd.tcl` 脚本是从现有的设计中提取的，使用 `write_bd_tcl -no_ip_version` 命令就可以完成这一操作。

   <img src="1.png" style="zoom:80%;" />

   <img src="2.png" style="zoom:80%;" />

### 2. 在模块设计中创建层次结构

在本节中，我们将在 BD 中创建两个 RP 的层次结构实例。在 BD 中，为 RP 创建一个层次结构（Hierarchy）是必要的，因为 RP 在后续中会被转换为一个 BDC，内部的逻辑将与顶层静态设计解耦。

工程目录中的 `create_rp1_bdc.tcl` 可以自动化地完成第一个 RP 层次结构的创建，这里我们在 GUI 中进行展示，具体步骤如下：

1. 在 BD 中右键点击 `axi_gpio_1` 实例，然后选择 Create Hierarchy。

2. 在弹出的对话框中，将层次结构命名为 `rp1`，然后点击 OK。

   <img src="3.png" style="zoom:80%;" />

3. 选择 `xlconstant_1` 实例，并将其拖放到 `rp1` 层次结构实例中。

4. 右键点击 AMD Zynq™ UltraScale+™ MPSoC 实例，然后选择 Create Hierarchy。

5. 在弹出的对话框中，将层次结构命名为 `static_region`，然后点击 OK。

   <img src="4.png" style="zoom:80%;" />

6. 逐个选择 BD 中剩余的实例（除了 `rp1`），将它们拖入层次结构 `static_region` 中。生成的 BD 可以通过 Regenerate Layout 重新整理显示布局。最终生成的 BD 应该如下图所示。

   <img src="5.png" style="zoom:80%;" />

7. 在画布上右键单击以选择 Validate Design，然后保存 BD。



### 3. 创建一个模块设计容器

在我们为 `rp1` 创建了层级之后，我们就可以将其转换为 BDC，以便后续为其创建 RP。创建 BDC 的具体步骤如下：

1. 在 BD 中右键点击 `rp1` 实例，然后选择 Create Block Design Container。

2. 将 BDC  命名为 `rp1rm1`，然后点击 OK。

   <img src="6.png" style="zoom:80%;" />

   此操作会将选择的层次结构转换为 BDC。其中包含的（子级）BD，将被标记为 `rp1rm1.bd`，容器的图标中包含一个看起来像六个矩形组成的金字塔图标。

   <img src="7.png" style="zoom:80%;" />

   同时，在 Sources 窗口中，可以看到新的 BD 设计已经被添加到项目中。

   <img src="8.png" style="zoom:80%;" />

   上述操作为 `rp1` 子模块创建了一个新的 BD。如果你在 `design_1` 块设计中展开 `rp1` 实例，你会发现你不能在该级别编辑设计。这是一个只读副本，因此要编辑设计，你必须从源视图中打开 `rp1rm1.bd` 块设计。

3. 通过选择顶层模块设计的地址编辑窗口来修改 `rp1rm1` 实例的地址空间。完全展开 Network 0 的信息，然后通过将其更改为 64K 来修改 `/rp1/axi_gpio_1/S_AXI` 的范围。

   <img src="9.png" style="zoom:80%;" />

4. 返回到图表，右键单击并选择 Validate Design。验证完成后，单击 Save 以保存 BD。

   目前的架构仍然是一个标准的 IP 集成器项目，拥有两个层次化的 BD。IP 集成器中的 BDC 功能允许您为 `rp1` 层次实例添加多个设计源文件，通过使用多个设计版本进行更改，或通过与团队成员共享子模块 BD 来进行团队设计。



### 4. 启用 DFX 功能

在本节中，您将在 IP 集成器中启用 DFX 功能，并在 `rp1` BDC 中添加新的 RM。

工程目录中的 `enable_dfx_bdc.tcl` 脚本可以自动化地为 BDC 启用 DFX 功能，这里我们在 GUI 中进行展示，具体步骤如下：

1. 在 Vivado IDE 中选择 Tools > Enable Dynamic Function eXchange 以启用 DFX 功能。在打开的对话框中选择 Convert 选项。

   一旦执行此步骤，您将看到新的菜单项出现，尤其是在 Flow Navigator 中的 DFX 向导和 Tools 菜单下。

   注意，这个转换步骤是不可逆的。

2. 在 `design_1` 图中，双击 `rp1` 实例以编辑 BDC。

3. 在 General 选项卡下，勾选此容器上的 Enable Dynamic Function eXchange 和 Freeze the boundary of this container 选项。

   <img src="10.png" style="zoom:80%;" />

   第一个操作会将 `rp1` 实例定义为 RP，第二个操作可以防止参数跨边界接口传播。

4. 点击 Addressing 选项卡以查看该 BDC 的地址空间。可以看到，地址偏移量是 0xA000_000，地址范围是 64K，与 `rp1rm1` 中提供的信息相匹配。勾选 Show Detailed View 以查看 rp1rm1 的地址范围与 rp1 的整体地址范围相匹配。

   <img src="11.png" style="zoom:80%;" />

5. 点击 OK 以保存更改，并返回到 `design_1` 图。

   此时可以看到，`rp1` 容器上的图标已经显示“DFX”标签。

   <img src="12.png" style="zoom:80%;" />

6. 点击 Validate Design，然后点击 Save 以保存设计。



### 5. 添加新的可重构模块

下面我们将为现有的 RP 创建新的 RM。

工程目录中的 `create_rp1rm2.tcl` 脚本可以自动化地创建并添加 RM，这里我们在 GUI 中进行展示，具体步骤如下：

1. 在 BD 中右键点击 `rp1` 实例，并选择 Create Reconfigurable Module。在打开的对话框中，给 RM 命名为 `rp1rm2`，然后点击 OK。

   一个新的 BD 将被创建并打开。图中包含三个输入引脚，与 `rp1` 分区第一个 RM 的端口列表相同。对于给定的 RP，每个 RM 的端口列表必须相同，即使不是所有端口都被每个 RM 使用。请注意，在日志（和脚本）中， `create_bd_design` 命令使用 `-boundary_from_container` 选项，从 BDC 中复制显式的端口列表。

2. 通过点击+图标并使用搜索栏查找 AXI GPIO IP，将新的 IP 添加到画布中。将其添加到画布后，双击以进行自定义。对于 GPIO，勾选所有输入框，确保 GPIO 宽度设置为 32，然后点击确定返回画布。

   <img src="13.png" style="zoom:80%;" />

3. 再次点击+图标，并使用搜索框将一个 Constant IP 添加到画布上。双击进行自定义。将 Const Width 改为 32，Const Val 改为 0xFACEFEED。点击确定以接受更改。

   <img src="14.png" style="zoom:80%;" />

4. 连接各个 IP 的引脚，如下图所示。

   <img src="15.png" style="zoom:80%;" />

5. 切换到 Address Editor 选项卡并注意尚未分配地址。右键点击 /axi_gpio_0/S_AXI 并选择分配。这将设置一个从地址 0x4000_0000 开始的 64K 范围。

6. 修改 Master Base Address 使其从 0xA001_0000 开始，然后保持范围为 64K。

7. 验证并保存 `rp1rm2` 。

   在这个简单设计中，`rp1rm1` 和 `rp1rm2` 之间只有两个不同之处：S_AXI 基地址不同；通过 GPIO 读取的常数值不同。第一个差异将用于展示 RM 之间可能存在不同需求的设计中必须创建和管理设备树覆盖。第二个差异将用于确认硬件中的动态重构已成功完成。



### 6. 确认所有可重构模块的地址空间（Aperture）

确保每个 RM 都为其 AXI 从接口具有适当的地址空间（偏移+范围），并与顶层设计中的实例对齐。这通常由 Vivado 自动完成，也可以进行手动设置，以下是具体步骤：

1. 在顶层 BD 中，双击 rp1 块设计容器。切换到 Addressing 选项卡并点击 Show Detailed View 复选框。

   <img src="16.png" style="zoom:80%;" />

   可以看到，S_AXI 端口的 `rp1` 的整体地址空间从地址 0xA000_0000 开始，范围总共为 128K。这是从 BDC 中的每个设计源文件收集地址信息并总结每个模块的需求自动计算的。

   如果地址空间必须扩展以包含尚未创建的新 RM，请将模式从自动切换到手动并编辑主偏移量或范围。

2. 点击 OK 返回到顶层 BD。

3. 验证并保存 `design_1.bd`。



### 7. 创建封装并为顶层模块设计生成运行目标

在进行综合与实现之前，需要为 BD 生成一个 HDL 封装（Wrapper）。

工程目录中的 `run_impl.tcl` 脚本可以自动化地生成封装并进行综合，这里我们在 GUI 中进行展示，具体步骤如下：

1. 在 Source 窗口中，右键单击 `design_1.bd` 并选择创建 HDL 封装器。保持 Let Vivado Manage 选项被选中，然后点击 OK。

   可以看见，在 Source 窗口中，已经创建了 `design_1_wrapper.v` 并被添加到项目中。此 HDL 文件实例化了 `design_1` 模块。

2. 在 Flow Navigator 中，点击 IP Integrator 下的 Generate Block Design 命令。在弹出的对话框中，保持选择 Out of context per IP 选项，然后点击 Generate。

   此操作为 `design_1` 中的每个 IP 创建脱离上下文综合（Out-of-Order Synthesize，OOC Synthesize）的运行。在 Design Runs 窗口中，您将看到包含在  `design_1`  中的所有 IP 的综合运行列表（除了 `rp1` 之外的内容）已被创建并正在运行。对于 BDC 内的 IP，对于源文件 rp1rm1 和 rp1rm2，已被创建但未运行，将在后续请求。

