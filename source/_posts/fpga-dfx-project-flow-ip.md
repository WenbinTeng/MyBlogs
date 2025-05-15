---
title: fpga-dfx-ip-project-flow
date: 2025-05-09 08:58:17
tags:
typora-root-url: fpga-dfx-project-flow-ip
---

# FPGA DFX IP Project Flow

在大型项目中，通常使用 IP 封装来模块化管理设计源代码，从而更方便地组织功能模块与模块间的互联关系。Vivado 从 2021.2 版本开始增强了基于 IP 的项目设计功能，使用户可以更直观地在模块设计（Block Design，BD）中使用 Dynamic Function eXchange（DFX）技术。

基于 IP 项目的 DFX 设计具备以下特性：

- 可在 BD 中创建模块设计容器（Block Design Containers，BDC）并自动识别其层次结构；
- 支持将 BDC 定义为可重构分区（Reconfigurable Partitions，RP）；
- 为每个 RP 创建一组可重构模块（Reconfigurable Modules，RM）；
- 自动生成顶层模块与子模块的综合运行、实现运行、配置定义及比特流生成，与基于 RTL 的项目流程保持一致。

在开始实验前，请先从 Xilinx 官网下载官方 DFX 教程设计文件：

[下载链接](https://www.xilinx.com/member/forms/download/design-license.html?cid=03e48cb4-ba89-496d-a3d7-cbaa2302ef79&filename=ug947-vivado-partial-reconfiguration-tutorial.zip)

解压文件至本地具备写权限的目录，并进入 `ipi_bdc_dfx_zu` 子目录。本文将以 ZCU102 开发板作为目标平台。尽管所有编译步骤也可通过 `run_all.tcl` 脚本一次性执行，本次演示将使用 Vivado GUI 中的 IP Integrator 完成设计流程，以更清晰地呈现基于 IP 项目的 DFX 使用方式。



### 1. 在 IP 集成器中创建设计

1. 启动 Vivado IDE。在 Tcl 控制台中，导航到 `ipi_bdc_dfx_zu` 目录。

2. 执行第一个 Tcl 脚本以创建 ZCU102 平台上的基础 BD 设计：

   ```tcl
   source create_top_bd.tcl
   ```

   该脚本将自动执行以下任务：

   - 为 ZCU102 目标开发板创建一个新工程；
   - 添加并自定义一组 IP；
   - 在 BD 中连接 IP 之间的接口；
   - 验证设计并保存 BD 设计。

   该脚本源于已有设计，通过 `write_bd_tcl -no_ip_version` 命令导出而成。

   <img src="1.png" style="zoom:80%;" />

   <img src="2.png" style="zoom:80%;" />



### 2. 在模块设计中创建层次结构

本节将手动在 Block Design 中创建两个 RP 所对应的层次结构实例。Vivado 要求 BD 中的 RP 必须定义在一个独立的层次结构（Hierarchy）之中，以便后续转换为 BDC，实现与静态区域的物理与逻辑隔离。

工程目录中的 `create_rp1_bdc.tcl` 脚本可自动完成下述操作，此处我们使用 GUI 进行展示：

1. 在 BD 画布中，右键点击 `axi_gpio_1` 实例，选择 *Create Hierarchy*。

2. 在弹出窗口中将层次命名为 `rp1`，点击 *OK*。

   <img src="3.png" style="zoom:80%;" />

3. 选择 `xlconstant_1` 实例，并将其拖放到 `rp1` 层次结构中。

4. 右键点击 Zynq® UltraScale+™ MPSoC 实例，选择 *Create Hierarchy*。

5. 在弹出的对话框中，将层次结构命名为 `static_region`，然后点击 *OK*。

   <img src="4.png" style="zoom:80%;" />

6. 依次将除 `rp1` 外的所有剩余模块拖入 `static_region` 中。可使用 *Regenerate Layout* 自动整理画布布局。最终生成的 BD 应该如下图所示。

   <img src="5.png" style="zoom:80%;" />

7. 在画布空白区域右键，选择 *Validate Design* 校验连接合法性。验证通过后保存 BD。



### 3. 创建一个模块设计容器

在完成 `rp1` 层次结构的构建后，我们可以将其转换为 BDC。BDC 是 Vivado IP Integrator 提供的逻辑封装机制，允许将该层次定义为一个 RP，并通过多个 BD 实例（即多个 RM）实现模块替换。

创建 BDC 的步骤如下：

1. 在 Block Design 中右键点击 `rp1` 实例，选择 *Create Block Design Container*。

2. 在弹出的对话框中将 BDC 命名为 `rp1rm1`，然后点击 *OK*。

   <img src="6.png" style="zoom:80%;" />

   此操作会将所选层级结构转换为一个 BDC，内部将自动生成一个子级 BD 文件，命名为 `rp1rm1.bd`。容器的图标中会包含一个看起来像六个矩形组成的金字塔图标。

   <img src="7.png" style="zoom:80%;" />

   你还将在 Sources 窗口中看到新的 BD 文件已添加至工程中，`rp1rm1.bd` 的条目会显示在 `design_1.bd` 的子节点中。

   <img src="8.png" style="zoom:80%;" />

   注意，BDC 创建完成后，原 `rp1` 层次结构在 `design_1` 中将变为只读状态，无法直接编辑。要修改其中的设计内容，必须从 Sources 窗口中打开并编辑 `rp1rm1.bd`。

3. 打开顶层模块设计，切换到 *Address Editor* 选项卡。展开 Network 0 信息，找到 `/rp1/axi_gpio_1/S_AXI`，修改其地址范围为 64K，以匹配后续 RM 的接口定义。

   <img src="9.png" style="zoom:80%;" />

4. 返回设计图视图，右键点击空白处，选择 *Validate Design* 以重新验证设计连接。验证通过后点击 *Save* 保存更新。




### 4. 启用 DFX 功能

完成 BDC 的创建后，我们将启用 DFX 功能，并为其定义 RP 及其属性。

工程目录中的 `enable_dfx_bdc.tcl` 脚本可自动完成以下操作，本文以 GUI 方式演示手动流程：

1. 在 Vivado IDE 中，点击菜单栏 *Tools → Enable Dynamic Function eXchange*。在弹出的对话框中选择 *Convert*。

   可以看到，在 Flow Navigator 左侧新增 *Dynamic Function eXchange Wizard*，在 Tools 菜单中新增与 DFX 相关的配置入口。

   注意，这个转换步骤是不可逆的，请在执行这个功能之前做好备份。

2. 在 `design_1` 的画布中，双击 `rp1` 实例以编辑 BDC。

3. 在 *General* 选项卡中，勾选以下两个选项：

   - *Enable Dynamic Function eXchange*：启用该容器的 DFX 功能，将其标识为 RP；
   - *Freeze the boundary of this container*：冻结容器边界，防止参数通过边界接口传播。

   <img src="10.png" style="zoom:80%;" />

4. 在 *Addressing* 选项卡中，检查容器的地址空间是否与 `rp1rm1` 的设置相匹配。可以看到，地址偏移量是 0xA000_000，地址范围是 64K，与 `rp1rm1` 中提供的信息相匹配。勾选 *Show Detailed View*，可以展开看到 `rp1rm1` 内部模块的地址空间布局应完全位于该地址范围内。

   <img src="11.png" style="zoom:80%;" />

5. 点击 *OK* 保存设置并关闭属性窗口。

   此时，你会看到 `rp1` 容器的图标已经显示 “DFX” 标签，表示该模块已被标记为一个 RP。

   <img src="12.png" style="zoom:80%;" />

6. 在 `design_1` 画布中右键空白区域，选择 *Validate Design* 进行结构校验，并点击 *Save* 保存设计。




### 5. 添加新的可重构模块

本节将为当前工程中的 `rp1` 分区添加一个新的 RM 模块。

目录中的 `create_rp1rm2.tcl` 脚本可自动完成以下操作，本文将手动演示 GUI 流程：

1. 在 `design_1.bd` 的画布中右键点击 `rp1` 实例，并选择 *Create Reconfigurable Module*。在弹出的对话框中，将新模块命名为 `rp1rm2`，点击 *OK*。

   Vivado 会自动创建一个新的 BD 文件 `rp1rm2.bd`，并将其标记为 `rp1` 分区的另一个 RM。在这个 BD 中可以看到三个输入引脚，与 `rp1` 分区第一个 RM 的端口列表完全一致。对于任意 RP，其下属所有 RM 必须具备相同的接口结构（端口名称、宽度与方向完全一致），即使部分端口在某些 RM 中未被使用。Vivado 会使用 `create_bd_design` 命令的 `-boundary_from_container` 选项，从 BDC 中复制端口列表。

2. 点击+图标，在搜索栏中输入并添加一个 AXI GPIO IP，将其拖入画布并双击打开配置界面：勾选 *All Inputs* 选项，确保 GPIO Width 设置为 32，然后点击 *OK* 返回画布。

   <img src="13.png" style="zoom:80%;" />

3. 再次点击+图标，用同样的方式搜索并添加一个 Constant IP 到画布上，然后双击以自定义 IP：将 Const Width 改为 32，Const Val 改为 0xFACEFEED，然后点击 *OK* 返回画布。

   <img src="14.png" style="zoom:80%;" />

4. 连接各个 IP 的引脚，如下图所示。

   <img src="15.png" style="zoom:80%;" />

5. 切换至 *Address Editor* 选项卡，Vivado 会自动检测当前 BD 尚未分配地址。右键点击 `/axi_gpio_0/S_AXI`，选择 *Assign Address*，Vivado 将自动分配起始地址 0x40000000，大小为 64K。

6. 修改 *Master Base Address* 为 0xA001_0000，然后保持地址范围为 64K，以确保与 `rp1` 分区顶层地址布局保持一致。

   <img src="16.png" style="zoom:80%;" />

7. 在画布空白处点击右键，选择 *Validate Design* 以验证 BD，点击 *Save* 保存设计。

在这个设计中，`rp1rm1` 与 `rp1rm2` 之间存在两处有意设计的差异：

- AXI 地址基址不同（`0xA000_000` vs `0xA001_0000`）：用于为不同的 RM 生成独立的设备树配置；
- GPIO 输出常数不同（`0xFFFFFFFF` vs `0xFACEFEED`）：用于在硬件验证时判断模块是否已经正确重构。



### 6. 确认所有可重构模块的地址空间（Aperture）

为了确保系统运行时不会有地址冲突导致运行结果错误，必须检查并统一所有 RM 的地址空间（Aperture）分配，包括地址偏移与访问范围。Vivado 会根据 RM 内部 IP 的地址设置，自动计算 RP 顶层的地址范围，但用户仍可在需要时进行手动干预：

1. 在顶层 BD 中，双击 `rp1` 以编辑容器。切换至 *Addressing* 选项卡，勾选 *Show Detailed View*，展开地址映射细节。

   <img src="17.png" style="zoom:80%;" />

   Vivado 将显示从每个 RM 中提取的地址空间，并自动求并集计算出该 RP 的地址分配窗口。在本例中，S_AXI 端口中 `rp1` 的整体地址空间从地址 0xA000_0000 开始，范围总共为 128K。这是由于 `rp1rm1` 与 `rp1rm2` 分别占用了 64K 的地址空间。

2. 若计划添加更多 RM，或现有 RM 所需地址空间超出当前范围，可将地址模式从 *Auto* 改为 *Manual*，自行更改偏移和范围字段。

3. 点击 *OK* 应用地址修改，返回至顶层 BD。

4. 点击 *Validate Design* 检查地址一致性，然后最后点击 *Save* 保存设计。



### 7. 创建封装并为顶层模块设计生成运行目标

在进入综合与实现流程之前，需要为顶层 BD 生成 HDL 封装（Wrapper），并初始化对应的综合运行。

目录中的 `run_impl.tcl` 脚本可自动完成本阶段操作，但此处采用 GUI 操作方式逐步展示：

1. 在 Sources 窗口中，右键单击 `design_1.bd` 并选择 *Create HDL Wrapper*。在弹出的对话框中选择默认选项 *Let Vivado manage wrapper and auto-update*，然后点击 *OK*。

   Vivado 会自动为顶层 BD 创建一个 Verilog 封装文件 `design_1_wrapper.v`，该文件实例化了完整的 BD，并将其作为综合与实现的入口模块。

2. 在 Flow Navigator 中，点击 *IP Integrator > Generate Block Design*。在弹出窗口中确认选项 *Out of context per IP*，点击 *Generate* 开始执行。

   <img src="18.png" style="zoom:80%;" />

   此操作将对 `design_1` 中的所有 IP 实例创建脱离上下文综合（Out-of-Order Synthesize，OOC Synthesize）的运行。创建完成后，可以在 *Design Runs* 窗口中可看到：

   - 所有非 RP 模块（如 Zynq MPSoC、AXI Interconnect 等）均已生成对应的综合运行并开始执行；
   - 位于 RP 内部的 IP（例如 `axi_gpio_0`）不会立即运行，它们的综合将在配置执行时触发；
   - `rp1rm1` 与 `rp1rm2` 也被注册为 OOC 综合对象，但尚未启动，Vivado 将在配置实现阶段根据依赖触发其运行。



### 8. 使用 DFX 向导定义配置

Vivado 提供了 DFX 向导（Dynamic Function eXchange Wizard），用于图形化创建配置组合、实现运行、静态锁定及子配置依赖管理。

完成顶层封装与 IP 综合完成后，下一步是定义 RMs 与 RPs 之间的映射关系，即配置（Configurations）。而配置运行（Configuration Run）是布局布线工具的一次运行，用于为该配置创建一个布线设计检查点（Design Check Point，DCP）。

以下是定义配置与运行的具体步骤：

1. 在 Flow Navigator 或 Tools 菜单中点击 *Dynamic Function eXchange Wizard* 以启动 DFX 向导，然后点击 *Next* 继续。

   <img src="19.png" style="zoom:80%;" />

2. 在 *Edit Reconfigurable Modules* 步骤中，Vivado 会自动识别 `rp1` 分区中的所有 RM：`rp1rm1_inst_0` 和 `rp1rm2_inst_0`。点击 *Next* 继续。

3. 在 *Edit Configurations* 步骤中，点击 *automatically create configurations* 以自动生成两个配置。若手动操作，也可点击+图标添加新配置，指定每个 RP 的 RM 映射。由于本项目仅包含一个 RP，该选项会创建两组配置，分别映射到 `rp1rm1` 和 `rp1rm2`。点击 *Next* 继续。

   <img src="20.png" style="zoom:80%;" />

4. 在 *Edit Configuration Runs* 步骤中，点击 *Standard DFX* 以为每个配置创建一个运行。

   <img src="21.png" style="zoom:80%;" />

   Vivado 将在父子配置之间共享锁定的静态设计：父运行中会实现静态设计，而子运行会重用这一静态设计实现结果。在本例中，父运行 `impl_1` 对应主配置 `rp1rm1`；子运行 `child_0_impl_1` 对应 `rp1rm2`，并依赖于 `impl_1` 的静态设计实现结果。点击 *Next* 继续。

5. 点击 *Finish* 以完成配置定义与运行结构建立。

   DFX 向导支持随时重新调用，可用于添加新的 RM、修改现有配置或更新实现运行结构。

   在 *Design Runs* 窗口中，可以看到新创建的 `child_0_impl_1` 运行。与 DFX 向导中的视图显示一致，子运行以缩进形式列于 `impl_1` 之后，表示其静态设计将复用 `impl_1` 的实现结果。

   <img src="22.png" style="zoom:80%;" />



### 9. 为可重构分区添加设计约束

在 DFX 设计中，每个 RP 都必须通过 Pblock 指定其布局规划（Floorplanning）。Pblock 是 Vivado 用于物理布局规划的边界约束区域，用于确保各个 RM 在实现过程中拥有一致的资源范围。

在本项目中，Pblock 约束已预定义于 `ipi_bdc_dfx_zu/constraints/pblocks.xdc` 文件中。

以下是添加约束的步骤：

1. 在 Sources 窗口中点击+按钮，打开 *Add Sources* 对话框。选择 *Add or Create Constraints*，然后点击 *Next*。点击 *Add Files*，定位至约束文件路径。选中该文件后点击 *Finish* 以将约束文件添加至项目中。

   如有需要，你可以打开综合后的设计来查看为此设计创建的 Pblock。



### 10. 实现配置并生成比特流

现在我们可以执行 DFX 的实现阶段。Vivado 会根据配置运行（Configuration Runs）中定义的父子关系，自动调度综合与布局布线任务，并在完成后生成相应的完整与部分比特流文件。

以下是实现的步骤：

1. 在 *Design Runs* 窗口中，右键点击子配置运行 `child_0_impl_1`，选择 *Launch Runs*，在确认对话框中点击 *OK* 开始执行。Vivado 会自动依照以下依赖顺序执行所有必要任务：
   - 对两个 RM 执行 OOC 综合，由于它们之间没有依赖关系，因此这些综合将并行启动。
   - 顶层设计的综合运行将在 RM 的 OOC 综合完成之后启动，由于顶层的静态设计只包含 I/O 连接和两个黑盒模块，因此综合完成得非常迅速。
   - 父运行 `impl_1` 将在子运行 `child_0_impl_1` 之前被执行。这个运行将按照标准 Vivado 实现流程进行布局与布线，并应用所有的约束文件（包括 Pblock），而且实现结束后将输出以下三类 DCP：
     - 完整的布线 DCP（包含静态设计与可重构模块）；
     - 单独的 RM （仅限 `rp1rm1`） 布线 DCP；
     - 仅包含静态设计 DCP，所有布局和布线都已锁定，其中的 RP 被置为黑盒。
   - 最后执行子运行 `child_0_impl_1`。其基于父运行中的静态设计 DCP 开始，加载新的 RM （即 `rp1rm2`）并实现。
2. Vivado 完成运行后将弹出提示窗口，点击 *Cancel*。此时可以查看实现后的设计。



至此，我们已经完成了基于 IP 项目的 DFX 设计，最终生成了适用于两种配置的完整比特流，以及对应于 `rm1` 和 `rm2` 的部分比特流。关于在 FPGA 设备上进行部分重构的操作，请参考 DFX 脚本流程。

