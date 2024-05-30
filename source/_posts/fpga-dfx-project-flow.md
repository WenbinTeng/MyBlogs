---
title: fpga-dfx-rtl-flow
date: 2024-05-23 13:56:09
tags:
typora-root-url: fpga-dfx-project-flow
---

# FPGA DFX Project Flow

在本次实验中，我们将创建一个基于 RTL 设计的 DFX 工程。实验的顶层模块将使用两个可重构模块来输出 LED 图案，用以说明分区定义的细节。

**1. 准备设计文件**

下载 Xilinx 提供的[参考设计文件](https://www.xilinx.com/member/forms/download/design-license.html?cid=03e48cb4-ba89-496d-a3d7-cbaa2302ef79&filename=ug947-vivado-partial-reconfiguration-tutorial.zip)，解压后进入 `\dfx_project` 文件夹，假设路径为 `<Extract_Dir>`。

**2. 加载工程和设计文件**

任何 DFX 设计流程（基于 Project 或 Shell 等）的第一个独特步骤是，定义设计中的可重构部分。这是通过项目模式下的分层源视图（Hierarchical Source View）中的上下文菜单完成的。以下步骤将通过在简单设计中定义分区，介绍项目的创建过程。

我们在 Vivado 中创建一个 RTL 工程，然后将以下存放有源文件的文件夹添加到工程中

- `<Extract_Dir>\Sources\hdl\top`
- `<Extract_Dir>\Sources\hdl\shift_right`

再将以下约束文件添加到工程中

- `<Extract_Dir>\Sources\xdc\top_io_<board>.xdc`
- `<Extract_Dir>\Sources\xdc\pblocks_<board>.xdc`

现在我们就创建了一个标准的 RTL 项目，没有任何 DFX 操作。

![](/1.png)

如要进行 DFX 操作，需要在导航栏中选择“Tools > Enable Dynamic Function eXchange”，这个操作不能撤销，请在这之前做好项目备份。在随后的对话框中，单击“Convert”将此项目转换为 DFX 项目。

![](/2.png)

右键单击“shift”实例，选择“Create Partition Definition”。此操作将在设计中将两个“shift”实例定义为可重构分区。因为每个实例都来自同一个 RTL 源文件，所以它们在逻辑上是相同的。如果您不希望一个源文件的所有实例都被定义为可重构的，那么需要手动地另外定义一个新模块。上下文无关综合（Out-of-context synthesis）将保持该可重构模块与顶层模块分离的模式运行，并且一个综合后检查点（Checkpoint，一般指 Vivado 的 .dcp 文件）将用于两个“shift”实例。

在出现的对话框中，为分区定义和可重构模块命名。分区定义是将所有可重构模块插入其中的工作空间的通用引用，因此请给它一个适当的名称，例如“shifter”。可重构模块指的是这个特定的RTL实例，所以给它起一个描述其功能的名字，比如“shift_right”，然后单击“OK”。

![](/3.png)

“Sources”窗口现在略有变化，两个“shift”实例现在都显示为黄色菱形，表明它们是分区。您还将在此窗口中看到“Partition Definitions”选项卡，其中显示设计中所有分区定义的列表和内容（此时只有一个）。此外，还创建了一个上下文无关模块运行来综合“shift_right”模块。

![](4.png)

此时，可以在 “Dynamic Function eXchange Wizard” 中操作以添加更多的可重构模块源文件。

**3. 使用动态功能交换向导完成设计**

在导航栏中进入 DFX 向导，点击“Next”进入“Edit Reconfigurable Modules”页面，可以看到“shift_right”可重构模块已经存在，我们点击“+”添加新的可重构模块，将以下文件夹添加到新模块中

- `<Extract_Dir>\Sources\hdl\shift_left`

如果有模块级的约束文件可以在这里添加，这里我们没有约束所以不作添加。将新的可重构模块命名为“shift_left”，可重构分区选择“shifter”，顶层模块字段保留为空，然后单击“OK”进行创建。现在“shifter”可重构分区下有两个可重构模块，单击“Next”继续。

![](5.png)

接下来选择“automatically create configurations” 来让 DFX 向导来自动创建可重构配置。选择此选项后，将创建两个配置的最小集，配置的名称可以自定义。使用这两个可重构模块可以创建额外的配置，但是这里只需要两个就可以创建这版设计所需的所有部分比特流，因为任何可重构分区的可重构模块的最大数量是两个（不包括灰盒可重构模块）。点击“Next”继续。

![](7.png)

与可重构配置类似，可以自动或手动创建用于实现每个配置的运行。运行之间如何交互将被视为父子关系——父运行实现静态设计和该配置中的所有可重构模块，然后子运行在已建立的上下文中实现该配置中的可重构模块时重用锁定的静态设计。选择“automatically create configuration runs”或者“Standard DFX”来自动创建运行。这将创建两次运行，由一个父配置（config_right）和一个子配置（config_left）组成。可以在此向导中创建任意数量的独立运行或相关运行，并为其中任何一个运行提供使用不同策略或约束集的选项。

![](9.png)

单击“Next”查看“Summary”页面，然后单击“Finish”完成设置并退出向导。

**4. 综合并实现现有设计**

在 IDE 中打开上述设计后，检查“Design Runs”窗口。顶层设计综合运行（synth_1）和父实现运行（impl_1）被标记为“active”。“Flow Navigator”应用于这些活动运行及其子运行，因此单击“Run Synthesis”或“Run Implementation”将设计应用于这些活动运行，以及完成它们所需的上下文无关综合运行。您也可以选择一个特定的父实现或子实现运行，右键单击并选择“Launch Runs”，来完成整个流程。

首先，在“Flow Navigator”中选择“Run Synthesis”来运行综合。综合运行将会首先综合上下文无关模块，然后再综合顶层模块。等待综合完成后，打开综合设计。在综合设计中我们可以看到已经定义的两个 Pblock，这是在约束文件 `pblocks_<board>` 中提供的，然后映射到两个 shift 实例。如果设计源代码中没有 Pblock，则可以在此步骤中创建它们。这可以通过右键单击设计层次结构中的“inst_shift”实例来选择“Floorplanning > Draw Pblock”来完成。每个实例都需要自己唯一的 Pblock。

![](11.png)

然后，通过选择“Reports > Report DRC”运行 DFX 特定的设计规则检查。为了节省时间，可以取消选中除“Dynamic Function eXchange”之外的所有复选框。DRC 会检查所提供的源文件和约束文件的报告没有错误。对于某些设备，可能会给出建议信息，用以提高 Pblock 的质量。对于这个简单的设计，这些可以忽略。

![](12.png)

接着，在“Flow Navigator”中，选择“Run Implementation”以在所有配置上运行布局布线。在本实验中，此操作首先运行“impl_1”的实现，然后运行“child_0_impl_1”的实现。除了本实验配置的两个运行的布局布线之外，Vivado 还会自动执行一些特定的 DFX 任务：

- 为每个布线的“shift_right”可重构模块写模块级别的上下文无关检查点。
- 在顶层模块中为每个可重构分区划分逻辑以创建静态的设计映像。这是通过为每个实例调用命令 `update_design -black_box` 来完成的。
- 锁定设计中静态部分的所有布局布线。这是通过调用 `lock_design -level` 布线完成的。
- 保存锁定的静态父映像以供所有子运行重用。

最后，在“Flow Navigator”中，点击“Generate Bitstream”。此操作在活动的父运行上启动比特流生成，并在所有实现的子运行上启动“PR Verify”，以确保设计图像的静态部分的一致性，再启动子运行的比特流生成。

