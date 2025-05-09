---
title: fpga-dfx-rtl-flow
date: 2024-05-23 13:56:09
tags:
typora-root-url: fpga-dfx-project-flow
---

# FPGA DFX RTL Project Flow

Vivado IDE 提供了完整的图形化流程来实现 Dynamic Function eXchange（DFX）功能。本文将通过一个基于 RTL 的工程，逐步展示如何在项目模式下使用 GUI 工具完成 DFX 设计流程。



### 1. 准备设计文件

从 Xilinx 官网下载官方 DFX 教程设计文件：

[下载链接](https://www.xilinx.com/member/forms/download/design-license.html?cid=03e48cb4-ba89-496d-a3d7-cbaa2302ef79&filename=ug947-vivado-partial-reconfiguration-tutorial.zip)

下载完成后，将文件解压至任意具备写权限的本地路径，然后进入解压后的 `dfx_project` 子目录。本实验中的所有步骤都将在该目录下完成。



### 2. 创建工程并加载设计文件

任何 DFX 设计流程（不管是基于 Project 还是 Tcl Shell）的首要步骤是，标记设计中可重构的部分。在 Vivado IDE 中是通过 Hierarchical Source View 中的上下文菜单完成的。以下操作将以一个简化的设计为例，说明如何创建并初始化一个带有可重构部分的 Vivado 工程：

1. 确保已正确提取 `dfx_project` 子目录。

2. 启动 Vivado IDE，点击 *Create Project*，然后点击 *Next*。

3. 设置工程路径为 `dfx_project` 目录。项目名称设置为 `project_1`，并勾选 *Create project subdirectory* 选项，然后点击 *Next*。

4. 选择 *RTL Project*，并取消勾选 *Do not specify sources at this time*，然后点击 *Next*。

5. 进入 *Add Sources* 页面，添加以下设计源文件路径，然后点击 *Next*。

   - `dfx_project/Sources/hdl/top`
   - `dfx_project/Sources/hdl/shift_right`

6. 进入 *Add Constraints* 页面，添加以下约束文件，然后点击 *Next*。

   - `dfx_project/Sources/xdc/top_io_<board>.xdc`
   - ``dfx_project/Sources/xdc/pblocks_<board>.xdc``

7. 在 *Default Part* 页面选择目标开发板，确保选择与约束文件匹配的器件型号。点击 *Next* 完成创建，此时 Vivado IDE 将加载 RTL 项目并显示层次结构视图。

   <img src="1.png" style="zoom:80%;" />

8. 在菜单栏中选择 *Tools → Enable Dynamic Function eXchange*。此操作将使该工程启用 DFX 功能，注意：此更改不可撤销，建议在启用前进行项目备份。随后点击 *Convert*，Vivado 会将当前项目转换为 DFX 项目格式。

   <img src="2.png" style="zoom:80%;" />

9. 在 Hierarchical Source 视图中，右键点击一个 `shift` 实例，选择 *Create Partition Definition*。Vivado 将识别两个 `shift` 实例属于相同 RTL 源，自动将它们映射为相同的可重构分区（Reconfigurable Partition，RP），并为其创建可重构分区。系统会自动对该模块执行一次脱离上下文综合（Out-of-Context Synthesis, OOC），并生成相应的 DCP。

10. 在弹出的对话框中，为分区和可重构模块（Reconfigurable Module，RM）指定名称。分区名称为该 RP 的逻辑标识（如 `shifter`），RM 名称通常采用模块功能名（如 `shift_right`）。完成后点击 *OK*。

    <img src="3.png" style="zoom:80%;" />

    现在 Source 窗口已经产生变化，两个 `shift` 实例都显示为黄色菱形，表示它们是分区。您还将在此窗口中看到 Partition Definitions 选项卡，其中显示设计中所有分区定义的列表和内容（此时只有一个）。此外，还创建了一个脱离上下文的运行（Run）来综合 shift_right 模块。

    此时，Source 视图中的两个 `shift` 实例将显示为黄色菱形图标，表示它们已被标记为 RP。此外，界面还将新增 *Partition Definitions* 标签页，用于管理所有已定义的 RP 和 RM 实例。Vivado 还会为初始 RM 启动一条 OOC 综合运行流程。

    <img src="4.png" style="zoom:80%;" />

    若需添加其他 RM，可使用 DFX 向导添加模块，后续将在下一节详细说明。



### 3. 使用 DFX 向导完成设计

Vivado 提供的 DFX 向导（Dynamic Function eXchange Wizard）可用于管理 RM 的添加、配置组合的定义、以及各实现运行的构建。在完成第一个 RM 的分区定义后，我们将通过该向导完成余下设置。

##### 启动 DFX 向导

在 IDE 中点击菜单栏 *Tools > Dynamic Function eXchange Wizard*，或在 Flow Navigator 中选择对应条目，点击 *Next* 进入向导。

##### 添加可重构模块

进入 *Edit Reconfigurable Modules* 页面时，已有的 `shift_right` 模块将自动显示。点击窗口左上角的蓝色 “+” 图标，添加新的 RM。在弹出的窗口中：

- 点击 *Add Directories*，选择目录：`dfx_project/Sources/hdl/shift_left`；
- 设置 RM 名称为 `shift_left`；
- 关联的分区选择为之前定义的 `shift`；
- 顶层模块字段保持为空；
- 保持 Synthesize sources 复选框未勾选。

点击 *OK* 完成添加。此时，`shift` RP 已拥有两个 RM 变体，点击 *Next* 继续。

<img src="5.png" style="zoom:80%;" />

##### 创建配置（Configuration）

进入 *Edit Configurations* 页面。配置定义表示完整系统镜像（Image），包括静态设计及其在各 RP 上的 RM 映射关系。我们可以在 DFX 向导 中创建任何所需的配置集，或者简单地让 DFX 向导自动选择。

这里选择 *automatically create configurations* 选项，DFX 向导将根据已有的 RM 自动生成最小配置集。执行此选项之后，Vivado 将创建拥有两个配置的最小集合，以包含所有的 RP 对应 RM 的映射组合。每个 `shift` 实例将在第一个配置中分配为 `shift_right`，在第二个配置中分配为 `shift_left`。每个配置的名称是可以修改的，如下图所示，配置的名称已经修改为 `config_right` 和 `config_left`，以反映每个模块中包含的可重构模块。

<img src="7.png" style="zoom:80%;" />

通过组合这两个 RM，最多可以生成四种不同的配置。但是 Vivado 只会为每个 RP 生成两个部分比特流文件，分别对应每个 RP 中的两个 RM，以最小化编译开销。点击 *Next* 继续。

##### 设置实现运行（Configuration Runs）

进入 *Edit Configuration Runs* 页面。与配置类似，用于实现每个配置的运行可以是自动创建或手动创建的。相关的运行之间将采用父子关系进行定义：父运行实现静态设计以及该配置中的所有 RM，然后子运行在已创建的上下文中重用锁定的静态设计，并在该配置中实现 RM。

对于 UltraScale+ 设备，请使用 *Standard DFX* 以用最小运行集填充配置运行页面，并支持使用 *Abstract Shell* 配置不同的运行。对于 7 系列或 UltraScale 设备，选择 *automatically create configuration runs* 选项，而且不支持使用 *Abstract Shell*  生成配置。

<img src="8.png" style="zoom:80%;" />

本实验选择 *Standard DFX*，Vivado 会自动构建两个运行，分别对应父配置 `config_right` 和子配置 `config_left` 。你可以在该页面中添加更多运行，或为任一运行设置特定的约束与综合策略。

<img src="9.png" style="zoom:80%;" />

点击 *Next* 查看汇总页面，确认设置无误后点击 *Finish* 完成 DFX 向导。

##### 查看设置结果

在 DFX 向导中更新设计之后，可以看到 *Design Runs* 标签中已更新状态。DFX 向导为 `shift_left` RM 添加了一个 OOC Synthesize 运行，并在父运行（`impl_1`）下创建了一个子运行（`child_0_impl_1`）。

<img src="10.png" style="zoom:80%;" />



### 4. 综合并实现设计

在配置好所有 RM 与配置后，即可进行综合（Synthesis）和实现（Implementation）阶段的编译工作。在 IDE 中，顶层设计的综合运行（`synth_1`）和父实现运行（`impl_1`）默认处于激活状态（“active”）。在 Flow Navigator 中发起的所有操作均针对这些激活的运行及其相关的 OOC 综合或子实现运行。你可以选择特定的父运行或者子运行，右键单击并选择 Launch Runs 以执行运行。以下是综合并实现设计的步骤：

##### 运行综合

在 Flow Navigator 中点击 *Run Synthesis*，在弹窗中确认后点击 *OK* 开始。Vivado 将首先综合所有 RM（OOC 运行），随后进行顶层设计的综合。完成后选择 *Open Synthesized Design* 打开综合结果。

在已综合的设计中，可在 Device 视图看到两个已定义的 Pblock 区域。这些 Pblock 来源于约束文件 `pblocks_<board>.xdc`，分别映射到两个 `shift` 实例。若未提供该约束文件，也可通过以下方式手动创建：在设计层次结构中右键单击 `inst_shift` 实例并选择 *Floorplanning > Draw Pblock*。

<img src="11.png" style="zoom:80%;" />

在 Device 视图中选择某个 Pblock，查看其属性，可以看到最后两个属性是 `RESET_AFTER_RECONFIG`（仅 7 系列）和 `SNAPPING_MODE`，这两个属性是 DFX 特有的。

点击 *Reports > Report DRC* 以执行检查，为了节省时间，可以取消选择除了 DFX 之外的所有复选框。

<img src="12.png" style="zoom:80%;" />

Vivado 将检查以下内容：

- 约束文件与实际资源之间是否一致；
- Pblock 是否正确对齐至时钟区域与合法列边界；
- RP 是否满足所需资源数量；
- 是否存在潜在的资源冲突或重构风险。

对于 7 系列设备，建议启用 `SNAPPING_MODE` 以自动修正边界对齐问题，特别是与 Clock Region 和 INT 列相关的错误。如果你创建了自定义的布局规划，并且报告了 DRC 问题，请在继续之前修复这些问题。

##### 运行实现

在 Flow Navigator 中点击 *Run Implementation*，Vivado 将自动依次完成所有配置的布局与布线。在这里，此操作首先为 `impl_1` 运行实现，然后为 `child_0_impl_1` 运行实现。在后台，Vivado 会处理以下细节：

- 为 `shift_right` RM 生成模块级别的（OOC）布线检查点；
- 使用 ``update_design -black_box`` 命令将每个 RP 替换为黑盒以提取静态设计；
- 使用 `lock_design -level routing` 命令锁定静态区域中的布局布线；
- 保存锁定后的静态设计检查点，供所有子运行复用。

若你仅希望实现其中一个配置运行，也可以在 *Design Runs* 窗口中单独选中目标运行右键执行。在启动子运行之前，必须成功完成父运行，因为子运行依赖于父运行的锁定静态设计。

##### 验证并生成比特流

当实现完成后，在弹出对话框中点击 *Cancel*。

<img src="13.png" style="zoom:80%;" />

现在，我们需要运行 PR Verify 来比较两个配置，以确保设计镜像中静态部分的一致性。这一步在 Vivado 中是默认执行的，也可以通过 `pr_verify` 命令来手动执行。

接着，我们为配置生成比特流文件。在 Flow Navigator 中，点击 *Generate Bitstream*。Vivado 会在激活的父运行中启动比特流生成，并在所有已实现的子运行中启动 PR Verify 和比特流生成。对于每个配置运行，默认情况下都会生成完整比特流和部分比特流。

至此，我们已经在 Vivado IDE 中的 GUI 中完成了所有 DFX 流程。此外，还有添加额外的可重构模块及其配置、创建和实现灰盒模块、修改设计源文件或选项等操作，详见官方文档。关于在 FPGA 设备上进行部分重构的操作，请参考 DFX 脚本流程。

