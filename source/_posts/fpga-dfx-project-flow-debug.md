---
*title*: fpga-dfx-project-flow-debug
date: 2025-05-10 08:40:57
tags:
typora-root-url: fpga-dfx-project-flow-debug
---

# FPGA DFX Debug for Project Flow

本篇博客将介绍如何在 Vivado 项目流程中为可重构模块（Reconfigurable Module，RM）插入调试核心，并利用 Vivado Hardware Manager 进行调试验证。



### 1. 准备设计文件

从 Xilinx 官网下载官方 DFX 教程设计文件：

[下载链接](https://www.xilinx.com/member/forms/download/design-license.html?cid=03e48cb4-ba89-496d-a3d7-cbaa2302ef79&filename=ug947-vivado-partial-reconfiguration-tutorial.zip)

下载完成后，将文件解压至任意具备写权限的本地路径，然后进入解压后的 `dfx_project_debug` 子目录。本实验中的所有步骤都将在该目录下完成。



### 2. 创建工程并加载设计文件

我们在 Vivado IDE 项目模式下的层次结构视图中表示设计的可重构部分，具体步骤如下：

1. 确保已正确提取 `dfx_project_debug` 目录。

2. 启动 Vivado IDE，点击 *Create Project*，然后点击 *Next*。

3. 设置项目路径为 `dfx_project_debug` 目录，设置项目名称为 `project_1`，并勾选 *Create project subdirectory* 选项，然后点击 *Next*。

4. 选择 *RTL Project*，并取消勾选 *Do not specify sources at this time*，然后点击 *Next*。

5. 进入 *Add Sources* 页面，添加以下源文件，然后点击 *Next*。

   - `dfx_project_debug/Sources/hdl/top.v`
   - `dfx_project_debug/Sources/hdl/multiplier/mult.v`
   - `dfx_project_debug/Sources/ip/<board>/clk_wiz/clk_wiz_0.xci`
   - `dfx_project_debug/Sources/ip/<board>/vio/vio.xci`

   注意不要选择 `add.v` 或 `mult_no_ila.v` 文件，这两个文件将在后续步骤中作为额外 RM 引入。

6. 勾选 *Copy sources into project* 复选框，点击 *Next*。

7. 进入 *Add Constraints* 页面，添加以下约束文件，然后点击 *Next*。

   - `dfx_project_debug/Sources/xdc/top_io_<board>.xdc`

8. 勾选 *Copy sources into project* 复选框，点击 *Next*。

9. 进入 *Default Part* 页面，选择目标开发板，确保选择与约束文件匹配的器件型号。点击 *Next* 完成项目创建。

10. 在 Vivado 中的 Sources 窗口中可看到设计层次结构（Hierarchy）。

    <img src="1.png" style="zoom:80%;" />

    如果某些 IP 显示红色锁标志，请通过 *Reports > Report IP Status* 检查其状态。如有需要，点击 *Upgrade Selected* 选项以更新 IP 到最新版本。

11. 在 *Flow Navigator* 中，展开 *Project Manager*，打开 *IP Catalog*，选择 *Debug & Verification > Debug*。

12. 右键点击集成逻辑分析仪（Integrated Logic Analyzer，ILA），并选择 *Customize IP*。在 *General Options* 选项卡以及 *Probe_Ports(0..0)* 选项卡中，自定义配置以下字段：

    - Component Name: ila_mult
    - Input Pipe Stages: 1
    - Probe Width of PROBE0: 8

    <img src="2.png" style="zoom:80%;" />

13. 点击 *OK*。Vivado 将自动将该 IP 插入至 `my_math` 层级下方。



### 3. 设置 DFX 功能

在插入 ILA 之后，即可为设计中的实例创建可重构分区（Reconfigurable Partition, RP）。具体步骤如下：

1. 依次点击菜单栏 *Tools > Enable Dynamic Function eXchange*，在弹出的对话框中点击 *Convert*，将当前项目转换为支持 DFX 的项目。

2. 在 Sources 窗口中右键点击 `my_math` 实例，选择 *Create Partition Definition*。

   此操作会将该模块标记为 RP，并自动执行一次脱离上下文综合（Out-of-Context Synthesis, OOC Synthesize），将其逻辑从顶层设计中剥离出来，以支持模块级替换。

3. 在弹出的对话框中，为分区定义和可重构模块命名。分区定义（Partition Definition）是工作区中对 RM 的统一引用，因此给它一个适当的名称： `math` 。可重构模块（Reconfigurable Module，RM）指的是这个特定的 RTL 实例，因此给它一个参考其功能的名称： `mult` 。

4. 点击 *OK*。

   <img src="4.png" style="zoom:80%;" />

   此时，`my_math` 实例在 Sources 窗口中将变为黄色菱形图标，表示该实例已被标记为 RP；在 *Partition Definitions* 标签页中会列出当前设计中所有 RP 与其已定义的 RM；Vivado 自动为 `mult` RM 创建了一个 OOC 综合运行。

   <img src="5.png" width="400" />

   <img src="6.png"  width="400" />

   你现在可以通过 DFX 向导添加其他 RM 或修改现有配置。



### 4. 使用 DFX 向导完成设计

Vivado IDE 中的 DFX 向导（Dynamic Function eXchange Wizard）用于集中管理 Reconfigurable Modules（RM）、配置（Configurations）与实现运行（Runs）。以下是使用 DFX 向导进行设计的具体步骤：

1. 在 *Flow Navigator* 或 *Tools* 菜单中点击 *Dynamic Function eXchange Wizard*，然后点击 *Next* 开始。

2. 进入 *Edit Reconfigurable Modules* 页面，可以看到已有的 `mult` RM。点击左侧的+按钮添加新的 RM。

3. 点击 *Add Files* 按钮并定位到源文件：

   - `dfx_project_debug/Sources/hdl/adder/add.v`

   若需要为该模块指定局部约束，可同时添加约束文件（要求限定于分区作用域）。

4. 点击 *OK* 添加源文件。

5. 在新的 RM 中：设置名称为 `add`；设置分区名称为 `math`；顶层模块名称留空；取消勾选 *Sources are already synthesized option* 选项；勾选 *Copy sources into project* 选项。单击 *OK* 以确认创建新模块。

   现在 `math` RP 中拥有两个 RM：`mult` 与 `add`。

   <img src="7.png" style="zoom:80%;" />

6. 点击 *Next* 进入 *Edit Configuration* 页面。

7. 选择 *automatically create configurations* 以让 DFX 向导自动创建配置。配置（Configuration）是包含静态设计和每个 RP 一个 RM 的完整设计镜像。你可以在 DFX 向导中创建任何所需的配置。

   现在 Vivado 已经自动创建了包含两个配置的最小集合，`math` 分区在两个配置中分别被分配了 `mult` 模块与 `add` 模块。

   <img src="8.png" style="zoom:80%;" />

8. 点击 *Next* 进入 *Edit Configuration Runs* 页面。

9. 选择 *Standard DFX* 或者 *automatically create configuration runs* 以自动创建最小运行集合。运行（Run）是实现配置的任务。在 DFX 向导中，你可以创建任意数量的独立或相关运行，并为它们中的每一个选择不同的策略或约束集。

   现在 Vivado 已经自动创建了两个运行，以实现一个父配置（`config_mult`）和一个子配置（`config_add`）。请注意，自动生成运行的名称是不可编辑的。
   
10. 点击 *Next* 进入 *Summary* 页面，并点击 *Finish* 以完成设置并退出 DFX 向导。在 Vivado IDE 中，可以看到，设计运行窗口已更新。Vivado 为 `math` RM 添加了第二个 OOC 综合运行，并在父实现运行（`impl_1`）下创建了一个子实现运行（`child_0_impl_1`）。



### 5. 在可重构模块中添加 IP

查看 Sources 窗口中的 *Partition Definitions* 选项卡，在 `math_rp` RM 下，看到其实例化了三个模块，但是这三个模块的源文件还没有添加到设计中，如下图所示。

<img src="11.png" style="zoom:80%;" />

这三个缺失的模块都是 IP。IP 实例必须在每个 RM 内部唯一，因此静态设计或另一个 RM 中不能使用相同的 ILA 核心实例。在 RM 中添加 IP 的具体步骤如下：

1. 右键点击 `mult` RM 下的 `ila_mult` 实例，然后选择 *Copy IP*。

2. 将 *Destination IP Name* 设置为 `ila_add`，并保持 *Destination IP Location* 不变，然后点击 *OK*。

   <img src="12.png" style="zoom:80%;" />

   复制完成后，Vivado 会将新 IP 放置在顶层设计中。

3. 在 *Hierarchy* 选项卡中，右键点击 `ila_add` 实例，选择 *Move to Reconfigurable Module*，然后选择 `add` 模块并点击 *OK*。

   <img src="13.png" style="zoom:80%;" />

   返回到 *Partitions Definitions* 选项卡，可以看到 ILA IP 实例已移动到 `add` RM 下。

4. 打开 *IP Catalog*，搜索 *Adder/Subtracter*，打开此 IP 并使用这些选项进行自定义，将名称设置为 `c_addsub_0`：

   - Input Type: Unsigned (for both A and B)
   - Input Width: 5 (for both A and B)
   - Output Width: 6
   - Latency: 0
   - 在 Control 选项卡中取消勾选 Clock Enable

5. 点击 *OK*。

   <img src="14.png" style="zoom:80%;" />

6. 点击 *Skip* 以生成 IP。

   与 ILA IP 类似，这个 IP 被放置到 Sources 窗口中的顶层设计层次结构中，因此请按照相同步骤将其移动到 `add` RM。

7. 在 *Hierarchy* 选项卡中，右键单击 `c_addsub_0` 实例，选择 *Move to Reconfigurable Module*，然后选择 `add` RM，然后点击 *OK*。



### 6. 综合设计并创建布局规划

由于本实验在 RM 中插入了调试 IP（如 ILA），Vivado 会自动添加调试专用的 Debug Hub 模块，因此需要在综合运行之前检查端口命名规范。以下是综合设计并创建布局规划的具体步骤：

1. 打开 `mult.v` 文件，检查端口定义。

   端口列表包括十二个以 `S_BSCAN_` 开头的端口。这些端口用于连接设计中静态部分和可重构部分中插入的 Debug Hubs。这些 Hubs 的插入是自动的，只要这些端口列表符合命名规范。

2. 在 Design Runs 窗口中，顶层设计的综合运行（`synth_1`）和实现运行（`impl_1`）被标记为已激活（active）。运行综合或者运行实现将会启动父子运行及其相关的 OOC 综合。你也可以右键单击子运行并选择 *Launch Runs* 来单独执行子运行。

   这里我们单独运行综合。在 *Flow Navigator* 中，点击 *Run Synthesis* 以运行综合。这个操作将首先执行所有模块的 OOC 综合，然后再进行顶层设计的综合。

3. 当综合结束之后，选择 *Open Synthesized Design* 以打开综合后的设计，查看原理图。在顶层设计中，我们看到 Vivado 已经插入了一个 `dbg_hub` 实例，它的 `sl_*` 端口连接到顶层 VIO 调试核心。进入 `my_math` 层次结构，可见另一个 `dbg_hub`，其 `sl_*` 接口连接至内部的 ILA。此时 RM 为 `mult`，你将在其图中看到连接路径。

   <img src="15.png" style="zoom:80%;" />

4. 点击顶部菜单 *Layout > Floorplanning* 进入布局规划模式。在 Netlist 窗口中，右键单击 `my_math` 实例并选择 *Floorplanning > Draw Pblock*。在设备视图中手动绘制一个 Pblock。在出现的对话框中，将绘制的 Pblock 命名为 `pblock_my_math`，并勾选 SLICE、DSP和BRAM资源类型。

   <img src="16.png" style="zoom:80%;" />

   如果绘制的 Pblock 中包含的资源不足，那么对应的资源类型会在 Pblock Properties 窗口的 *Statistics* 选项卡中显示为红色。在这种情况下，你调整 Pblock 大小，以包含足够的资源数量，然后保存这个布局规划。建议所绘制的 Pblock 至少包含 3000 个 CLB 与一个 BRAM 列。

5. 选择 *Reports > Report DRC* 以运行设计规则检查（Design Rule Checking，DRC）。为了节省时间，取消勾选除了 DFX 以外的所有复选框。然后点击 *OK* 以运行。

   <img src="17.png" style="zoom:80%;" />

   运行 DRC 后，请修复任何可能出现的错误，以继续执行后面的步骤。

6. 点击顶层工具栏中的 *Save Constraints* 来保存约束。

   

### 7. 运行 PR 配置分析报告

Vivado 提供了PR 配置分析（PR Configuration Analysis）工具来比较每个 RM，通过分析资源使用情况、布局规划、时钟和时序指标，判断当前 RP 对应的 Pblock 是否能够覆盖所有 RM 的最坏情况。

PR 配置分析工具通过 Tcl 控制台运行，具体操作如下：

1. 在 Tcl 控制台中，定位至工程目录，执行以下命令：

   ```tcl
   report_pr_configuration_analysis -cells my_math -dcps {./project_1.runs/add_synth_1/math_rp.dcp ./project_1.runs/mult_synth_1/math_rp.dcp}
   ```

   该命令将比较 `add` 和 `mult` 两个 RM 的综合结果，并生成一份综合分析报告，默认内容包含以下三个关键模块：

   - `-complexity`：报告资源使用复杂度，包括所有类型资源的最大占用；
   - `-clocking`：列出每个 RM 的时钟源、负载、扇出；
   - `-timing`：检查 Partition Pins 处的边界时序质量。

   你也可以添加可选项：

   - `-rent`：引入布线密度指标，但运行时间较长；
   - `-file`：指定输出文件路径以保存报告内容。

   在 Tcl 控制台中检查报告，可以在第二段中看到一个 Complexity 汇总。报告将列出 RM（当前配置）、RM1 与 RM2（被比较配置）以及一个资源“最大值”列；Vivado 会根据该分析建议你将 Pblock 尺寸设置为可容纳“最大资源使用”RM；若某个 RM 的资源计数为 0，则可能因 IP 未正确综合，导致分析不完整。

   请注意，如果 RM1 和 RM2 的资源计数似乎较低。在日志中的报告上方，你将看到一些关键警告：

   ```
   CRITICAL WARNING: [Project 1-486] Could not resolve non-primitive black box cell 'math_rp_c_addsub_0' instantiated as
    'adder_ip_instance0'
   ```

   这是因为 RM 中的 IP 以 OOC 模式存在，而 Vivado 尚未链接对应的 IP 综合结果。

2. 为保证分析报告完整，我们需要将 RM 内部的 IP 设置为全局综合模式（Global synthesis）：在 *Partition Definitions* 标签页中，右键点击 `math_rp_c_addsub_0`，选择 *Generate Output Products*。

3. 在弹出的设置窗口中，将 *Synthesis Options* 设置为 *Global*，点击 *Apply* 后点击 *Cancel*。

4. 对 `add` RM 下的两个 IP 执行相同操作：`my_add_ila` 和 `adder_ip`。后者有两个实例，但此操作只需要执行一次，因为该操作会自动应用到所有实例。

5. 修改之后，这些综合已经过期（out-of-date），在 *Flow Navigator* 中选择 *Run Synthesis* 以重新运行综合。在弹出的对话框中选择 *Accept*。

6. 综合完成后，重新运行 `report_pr_configuration_analysis` 命令，并检查日志和结果。



### 8. 实现设计

完成综合与布局规划后，即可启动 Vivado 的实现流程，具体步骤如下：

1. 在 *Flow Navigator* 中，选择 *Run Implementation* 以运行配置上的布局布线。

   这个操作会首先为 `impl_1` 运行实现，然后为 `child_0_impl_1` 运行实现。除了运行这两个运行的布局布线任务之外，Vivado 还会自动执行以下特定于 DFX 的任务：

   - 为布线后的 `mult` RM 生成模块级 DCP。
   - 调用 `update_design -black_box` 命令，将 RP 中的逻辑替换为黑盒。
   - 调用 `lock_design -level routing` 命令，锁定静态设计中的所有布局布线。
   - 保存锁定的静态设计 DCP 以供子运行使用。

   此外，当子运行完成后，Vivado 会为 `mult` RM 单独生成 DCP。而且子运行不需要生成包含静态设计的 DCP，因为这与父运行的内容一致。

   Vivado 会自动管理运行依赖关系。如果源文件（如 `add.v`）发生变更，对应的 OOC 综合运行和子实现运行会被标记为过期。

   如果只需要特定的配置运行，可以在 Design Runs 窗口中单独选择。请注意，子运行必须在父运行成功完成之后才能启动，因为子运行是通过从父运行导入锁定的静态设计 DCP 来启动的。

   <img src="18.png" style="zoom:80%;" />

2. 实现运行完成后，选择 *Open Implemented Design*，并在弹出的对话框中点击确定。

   <img src="19.png" style="zoom:80%;" />

   <img src="20.png" style="zoom:80%;" />

   上图展示了 `mult` 模块的布线设计。

3. Vivado 为每个 Pblock 提供一组可视化脚本，帮助你查看 RP 使用的布局帧以及实际布线使用范围。在 Tcl 控制台中，定位到工程目录，运行以下可视化脚本：

   ```tcl
   source project_1.runs/impl_1/hd_visual/pblock_my_math_Routing_AllTiles.tcl
   highlight_objects -color yellow [get_selected_objects]
   ```

   此脚本将用黄色高亮显示可用于布线的物理帧，覆盖整个时钟区域高度并在水平方向上扩展两个可编程单元列。

4. 运行以下可视化脚本：

   ```tcl
   source project_1.runs/impl_1/hd_visual/pblock_my_math_Placement_AllTiles.tcl
   highlight_objects -color blue [get_selected_objects]
   ```

   此脚本将用蓝色高亮显示实际逻辑布局帧，通常略小于布线范围。

   现在的设备视图应该看起来如下图所示，其中突出显示的 `math` RP 显示蓝色的布局区域、黄色的布线边界。

   <img src="21.png" style="zoom:80%;" />

   静态逻辑可以放置在扩展的布线区域中，即现在显示为黄色的剩余布线区域。

5. 在 *Flow Navigator* 中，选择 *Report Timing Summary*，然后点击 *OK* 以分析设计时序。

6. 在 *Timing* 选项卡中，选择 *Design Timing Summary*，并点击 *Worst Negative Slack (WNS)* 以显示前十个最差时序的关键路径。双击第一个路径以打开对应的时序摘要。

   在时序报告中，你将看到新的 Partition 列标示路径上的分区边界信息。这有助于识别跨 RP 的关键路径是否受限。

   <img src="22.png" style="zoom:80%;" />

7. 选择 *File > Close Implemented Design* 以关闭 `impl_1 ` 设计。



### 9. 添加额外的可重构模块和相应配置

本节将添加第三个 Reconfigurable Module（RM）：`mult_no_ila`，该模块功能与 `mult` 相同，但未包含 ILA 调试核心。尽管调试逻辑被移除，为了保持 RM 间的边界一致性，仍需保留相同的端口接口结构。以下是具体步骤：

1. 打开 DFX 向导。

2. 在 *Edit Reconfigurable Modules* 页面，点击+按钮添加一个新的 RM。

   <img src="23.png" style="zoom:80%;" />

3. 添加源文件 `dfx_project_debug/Sources/hdl/multiplier_without_ila/mult_no_ila.v`，将 RM 命名为 `mult_no_ila`，点击 *OK*，然后点击 *Next*。

   <img src="24.png" style="zoom:80%;" />

4. 进入 *Edit Configurations* 页面，点击+按钮，创建新的配置，将其命名为 `config_mult_no_ila`。为新配置中的 RP 分配 `mult_no_ila` RM。点击 *Next*。

   <img src="25.png" style="zoom:80%;" />

5. 进入 *Edit Configuration Runs* 页面，点击+按钮，创建新的配置运行，并设置以下参数：

   - Run: child_1_impl_1
   - Parent: impl_1
   - Configuration: config_mult_no_ila

   点击 *OK* 以接受此新配置。

   <img src="26.png" style="zoom:80%;" />

   这个新配置作为现有 `impl_1` 的子配置，将重用静态设计的实现结果。现在有三个运行，其中两个是子配置。绿色的勾表示有两个运行目前已完成。

   <img src="27.png" style="zoom:80%;" />

6. 点击 *Next*，然后点击 *Finish* 以构建这个新的配置运行。

   <img src="28.png" style="zoom:80%;" />

7. 右键点击这个新的子实现运行，然后选择 *Launch Runs*。这会在 `mult_no_ila` 模块上运行 OOC 综合，然后在锁定的静态设计上下文中实现这个模块。

8. 在实现完成后，在弹出的对话框中选择 *Cancel*。

   右键单击 `child_1_impl_1 ` 并选择 *Open Run*，可以在设备视图中观察到：

   - 静态逻辑被锁定，显示为橙色。
   - 在 *Design Runs* 选项卡中，可以看到 Pblock 中的逻辑量比其他配置要小得多，原因是没有实例化 ILA 调试核心。

9. 选择 *Tools > Schematic* 以打开原理图，进入 `math_rp` 实例，可以看到所有 BSCAN 端口都连接到了 LUT，并且没有插入 ILA 或 Dbg_Hub 核心。



### 10. 生成比特流

完成所有配置的综合与实现后，即可生成对应的比特流。在生成比特流之前，Vivado 会默认执行 PR Verify 以比较两个配置，确保 DCP 中静态部分的一致性。以下是生成比特流的具体步骤：

1. 在 *Flow Navigator* 中，点击 *Generate Bitstream*。Vivado 将自动进行：
   - 在激活的父运行中启动比特流生成；
   - 在已实现的子运行中运行检查然后生成比特流；
   - 为每个配置运行生成完整比特流和部分的比特流。



### 11. 连接并烧写 FPGA 开发板

本节将使用 Vivado 硬件管理器（Hardware Manager）加载完整和部分比特流文件，验证各 RM 在运行时的功能变化，并结合 VIO 和 ILA 调试核心进行交互式控制与波形观测。以下是具体步骤：

1. 在硬件管理器中连接到目标开发板。

   连接的硬件既可以是本地 FPGA 开发板或者是远程 FPGA 服务器。

2. 连接硬件后，右键单击 FPGA 实例并选择 *Program Device*。默认情况下，应从 `project_1.runs/impl_1` 目录中加载 `top.bit` 文件，同时 Vivado 会自动选中对应的 `top.ltx` 探针文件。

3. 在仪表板中点击 `hw_vio_1` 选项卡。

4. 点击按下+按钮，从 *Add Probes* 对话框中选择所有探针，然后点击 *OK*。

   <img src="29.png" style="zoom:80%;" />

5. 右键点击探针，并按照以下方式设置：

   - count_out_OBUF[7:0] bus – Radix: unsigned decimal
   - count_out_OBUF[7:0] individual bits – LED: low value Red, high value Green
   - pause_vio_out – Active High Button
   - reset_vio_out – Active High Button
   - toggle_vio_out – Active High Button
   - vio_select – Toggle Button

   配置完成后，仪表板应如下图所示：

   <img src="30.png" style="zoom:80%;" />

6. 将 `vio_select` 值设置为 1。这会禁用物理板上的按钮，并通过 VIO 启用暂停、重置和切换按钮。

7. 通过点击 `pause_vio_out` 的 Value 字段来使用暂停按钮。您将看到 LED 计数器停止在一个特定值。注意 `count_out_OBUF` 的无符号二进制值，在这个屏幕截图中，值是 12。

   <img src="31.png" style="zoom:80%;" />

8. 按下 `toggle_vio_out` 切换按钮。`count_out_OBUF` 总线的值将被平方，因为当前的 RM 是一个乘法器。在本例中，值是 144。

9. 再次按下暂停按钮，计数器将开始。`count_out_OBUF` 值现在将以 0 到 15 的平方进行计数。例如，1，4，9，16，25，等等。

10. 按下 `reset_vio_out` 重置按钮，将设计恢复到默认状态。计数器将恢复到初始状态（0~15 平方序列）。

11. 如果你有一个本地 FPGA 开发板，可以切换 `vio_select`，并使用开发板上的按钮和 LED 来观察相同的行为。

12. 切换到 ILA 仪表板。此时，你已经使用了静态设计中的 VIO，可以看到乘法器的结果，但如果您想观察 RM 内部的波形，您可以使用 ILA 来实现。

13. 在 Trigger Setup 窗口中，按下+按钮并添加 `my_math/mult[7:0]` 探针。更改数制为无符号十进制，并将值设置为 196（即 14x14）。

    <img src="32.png" style="zoom:80%;" />

14. 在 ILA 的设置窗口中，将触发位置更改为 512。

    <img src="33.png" style="zoom:80%;" />

15. 点击波形工具栏中的运行触发按钮，你会看到波形跳变。

    <img src="34.png" style="zoom:80%;" />

16. 现在加载 `add` RM的部分比特流。在 Hardware 视图中右键点击目标器件，并选择 *Program Device*。

17. 如果你要使用 UltraScale 器件，必须首先烧写用于清除的部分比特流，以准备下一个部分比特流的烧写。这里定位到 `project_1.runs/impl1/` 目录，并选择 `my_math_mult_partial_clear.bit` 文件用于清除对应区域。

    <img src="35.png" style="zoom:80%;" />

    Vivado 将会自动选择配对的 LTX 探针文件。点击 *Program* 以烧写新的部分比特流。

18. 切换到 VIO 仪表板，观察到计数器仍在计数。如果你按下切换按钮来切换到乘法器输出，可以看到值被保持在 255，这是因为 RP 中的逻辑目前被禁用。

19. 在 *Hardware* 视图中右键点击目标器件，并选择 *Program Device*。

20. 定位到 `project_1.runs/child_0_impl_1/` 目录，并选择 `my_math_add_partial.bit` 文件作为烧写的比特流。

    <img src="36.png" style="zoom:80%;" />

    再次地，Vivado 将会自动选择配对的 LTX 探针文件。点击 *Program* 以烧写。

21. 在 VIO 仪表板上，按下暂停按钮。此时，输出值为 6。按下切换按钮后，输出值为 18。加法器将相同的数字加 3 次。

22. 切换到 ILA 仪表板。在 Trigger Setup 窗口中点击+按钮，并添加 `my_math/outtemp2[5:0]` 总线到窗口中。更改触发器的以下设置：

    - Radix = [U]
    - Value = 30

23. 在 ILA 设置窗口中将触发器位置更改为 512。

24. 在波形窗口中，点击+按钮并将 `my_math/outtemp2[5:0]` 总线添加到波形图中。右键点击探针并将数制更改为无符号十进制。

25. 在波形窗口中，点击 ILA 的触发按钮。你可以看到从 27（9+9+9）到 30（10+10+10）的波形变换。

    <img src="38.png" style="zoom:80%;" />

    当确认一切运行正常时，关闭硬件管理器。

    

