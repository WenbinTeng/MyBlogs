---
title: fpga-dfx-abstract-shell
date: 2024-05-29 16:32:36
tags:
typora-root-url: fpga-dfx-abstract-shell
---

# FPGA DFX Abstract Shell

Vivado 允许使用上下文方法编译 DFX 设计，这个解决方案需要多次布局布线。首先，需要建立静态设计实现结果，以及每个可重构分区的第一个可重构模块。然后，所有后续的布局布线都在初始静态映像的上下文中完成。在实现第一个可重构模块之外的任何可重构模块之前，必须将包含整个静态区域的网表、布局布线信息的完全布线和锁定的静态设计数据库加载到 Vivado 中。

Abstract Shell 解决方案减少了对上下文工作流的需求。因为静态设计是锁定的，所以在实现新的可重构模块时不能修改它。上下文仍然是关键的，因此工具的工作路径没有改变。但是，工具不会加载一个完整的静态设计映像，而是使用一个 Abstract Shell 检查点。这个 Abstract Shell 只包含一个最小的逻辑和物理数据库，用于在特定的可重构分区中实现一个新的可重构模块，以验证时序并通过可重构分区验证，然后为该可重构模块生成部分比特流。

**1. 准备设计文件**

下载[参考设计文件](https://www.xilinx.com/member/forms/download/design-license.html?cid=03e48cb4-ba89-496d-a3d7-cbaa2302ef79&filename=ug947-vivado-partial-reconfiguration-tutorial.zip)，解压后进入 `abstract_shell/dfxc_vcu118/` 目录，假设路径为 `<Extract_Dir>`。

执行设计文件

**2. 执行设计流程**

从命令 shell 中，使用示例设计项目创建脚本启动 Vivado。

```
vivado –mode tcl –source project_dfxc_vcu118.tcl
```

脚本完成后，通过命令 start_gui 打开 Vivado IDE。检查 IP 是否需要更新。在 Flow Navigator 中，在 IP INTEGRATOR 标题下，单击 Generate Block Design 以准备处理设计。选择上下文无关的IP综合选项，然后单击 Generate。然后在 Design Runs 选项卡中，右键单击 impl_1 并选择 Launch Runs，这将只针对设计的父运行进行综合与实现。当 impl_1 完成时，在结果对话框中选择 Open Implemented Design。在 Vivado IDE 中打开实现后的父设计后，就可以创建 Abstract Shell 了。

**3. 创建 Abstract Shell**

默认情况下，在 DFX 工作流程中，多个设计检查点将在父配置的布局布线完成后写入。除了完整的设计布线检查点之外，Vivado 项目工作流还将为每个可重构分区调用 update_design -black_box，然后是 lock_design -level routing，从而创建一个静态的设计检查点，作为所有子运行的起点。此外，通过调用 write_checkpoint -cell 为父配置中的每个可重构模块编写模块级检查点。创建这些文件不需要用户干预。

在目录 `abstract_shell/dfxc_vcu118/project_dfxc_vcu118/project_dfxc_vcu118.runs/impl_1` 中是为父配置创建的文件，现在可以查看不同检查点的大小：

- top_routed.dcp (58,284 KB) – 全布线设计，包括每个可重构分区的一个可重构模块。
- top_routed_bb.dcp (55,819 KB) - 静态设计与锁定的布局布线和每个可重构分区的黑盒。
- u_count_count_up_routed.dcp (1,267 KB) - count_up 可重构模块实例的模块级布线检查点。
- u_shift_shift_right_routed.dcp (463 KB) – shift_right 可重构模块实例的模块级布线检查点。

考虑到这种设计中的大小和复杂性，可重构模块检查点比静态设计检查点小得多并不奇怪。下图显示了左侧的完整设计检查点和右侧的仅静态检查点。

![](1.png)

现在我们为为 u_count 和 u_shift 实例创建 Abstract Shell。确保 Tcl 控制台中当前的工作目录是 `<Extract_Dir>`，与 `project_dfxc_vcu118.tcl`、`sources` 和 `abstract_shell`文件夹所在的位置相同。执行以下命令：

```
write_abstract_shell -force -cell u_count ./abstract_shell/ab_sh_count.dcp
write_abstract_shell -force -cell u_shift ./abstract_shell/ab_sh_shift.dcp
```

每次调用 `write_abstract_shell` 会首先在内存中创建一个完整设计检查点的副本，然后自动运行以下步骤:

- 划分出目标可重构分区（使用 update_design -black_box）。
- 锁定剩余的设计，包括任何其他可重构模块。
- 为目标可重构分区写入 Abstract Shell。
- 与原始的完全布线设计相比，为此检查点运行 pr_verify。

这个过程比简单地调用 write_checkpoint 花费的时间要长，但是在几乎所有情况下，可重构模块编译的运行时节省的时间使得这一初始步骤是值得的。

现在查看 Abstract Shell 的大小，将它们与 top_routed_bb.dcp 完整 shell 检查点的大小进行比较：

- ab_sh_count.dcp (1,785 KB) – Count 可重构分区的 Abstract Shell。
- ab_sh_shift.dcp (1,699 KB) – Shift 可重构分区的 Abstract Shell。

通过以下命令打开每个 Abstract Shell 检查点来检查内容：

```
open_checkpoint ./abstract_shell/ab_sh_count.dcp
```

![](2.png)

您将看到每个 Abstract Shell 中只保留一个可重构分区（u_count 的 Shell 不包括 u_shift，反之亦然）。但是，即使绝大多数静态设计已被删除，部分设计仍然存在，包括来自 DFX 控制器和 DFX 去耦设计的元素，因为它们与每个目标可重构分区有连接。

![](3.png)

通过以下命令来运行布线报告以确认 Abstract Shell 是完整的。

```
report_route_status
```

这一步是可选的，仅仅表明 Abstract Shell 是一个有效的设计数据库，没有布线错误。

**4. 在 Abstract Shell 中实现新的可重构模块**

此时，所有剩余的可重构模块都可以在这些 shell 中实现。如果需要，每个可重构模块可以在单独的 Vivado 会话中并行实现，因为每个可重构分区可以独立管理。这不仅可以用于特定的 shell，例如 u_count 实例，还可以用于设计中的所有可重构分区。在项目流程中，重点放在完整的设计配置上，而在 Abstract Shell 方法中，重点放在可重构模块上。

现在打开 Vivado 创建一个新的对话，在 Tcl 控制台中，导航到解压目录。加载一个 Abstract Shell 检查点，而不是完整的静态检查点。

```
add_files ./abstract_shell/ab_sh_count.dcp
```

然后仅为 count_down 模块添加综合后的网表。

```
add_files ./project_dfxc_vcu118/project_dfxc_vcu118.runs/count_down_synth_1/count.dcp
set_property SCOPED_TO_CELLS {u_count} [get_files ./project_dfxc_vcu118/project_dfxc_vcu118.runs/count_down_synth_1/count.dcp]
```

当链接设计时，只使用 u_count 可重构分区。

```
link_design -mode default -reconfig_partitions {u_count} -part xcvu9p-flga2104-2L-e -top top
```

正常执行设计的实现，然后在保存布线设计的时候，保存完整的当前映像（ Abstract Shell 加上可重构模块）以及仅可重构模块的检查点。

```
opt_design
place_design
route_design
write_checkpoint -force ./abstract_shell/abs_count_down/abstract_shell_count_down_routed.dcp
write_checkpoint -force -cell u_count ./abstract_shell/abs_count_down/rm_count_down_route_design.dcp
```

对 shift_left 模块重复这些步骤，并相应地调整文件名和命令。可以在教程目录中找到用于 count_down 和 shift_left 的 Abstract Shell 实现的 Tcl 脚本，以自动执行这些步骤。脚本名称如下：

- `abs_impl_count_down.tcl`
- `abs_impl_shift_left.tcl`

在下图中，将 Count 可重构分区对应 Abstract Shell 中的 count_down 可重构模块与单独用于 count_down  可重构模块的模块级别检查点进行比较。只有前者会用于部分比特流生成，因为后者在动态区域中不包含静态设计信息。

![](4.png)

![](5.png)

通过以下脚本对两个检查点执行验证检查：

```
pr_verify ./abstract_shell/ab_sh_count.dcp ./abstract_shell/abs_count_down/abstract_shell_count_down_routed.dcp
```

