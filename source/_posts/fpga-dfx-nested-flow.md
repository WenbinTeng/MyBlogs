---
title: fpga-nested-dfx
date: 2024-05-29 09:10:29
tags:
typora-root-url: fpga-dfx-nested-flow
---

# FPGA DFX Nested Flow

本实验涵盖了一个简单的嵌套动态功能交换（Nested DFX）示例。嵌套 DFX 的概念是在一个动态区域内放置一个或多个动态区域，细分设备以允许更细粒度的重新配置。有了这个特性，您可以将一个可重构分区分割成更小的区域，每个区域都是部分可重构的。

本实验的设计是其他实验中使用的 LED-Shift-Count 设计的修改版本。不是简单地交换不同的移位器或不同的计数器，而是插入了一个额外的可重构层，使您能够在当前设计中拥有两个移位器或两个计数器。然后，这些移位器或计数器中的每一个都可以单独部分地重新配置。

嵌套 DFX 在工程模式中还不受支持，所以需要使用 Tcl 脚本解决方案。

**1. 准备设计文件**

下载[参考设计文件](https://www.xilinx.com/member/forms/download/design-license.html?cid=03e48cb4-ba89-496d-a3d7-cbaa2302ef79&filename=ug947-vivado-partial-reconfiguration-tutorial.zip)，解压后进入 `nested_dfx` 目录，假设路径为 `<Extract_Dir>`。

**2. 阐述脚本功能**

首先介绍综合脚本。文件 `run_synth.tcl`，`design_settings.tcl` 和 `advanced_settings.tcl` 位于根目录。`run_synth.tcl` 脚本包含运行此 DFX 设计的综合部分所需的最小设置，实现、验证和比特流生成是交互界面运行的。`design_settings.tcl` 脚本选择目标设备和单板，并为项目设置相对路径。`advanced_settings.tcl` 包含默认的工作流程设置，只能由有经验的用户修改。

接着介绍嵌套 DFX 相关脚本。在本实验中，在综合之后，您将使用单个 Tcl 命令逐步完成嵌套 DFX 工作流程。本实验旨在展示插入第二层可重构性所需的独特细节，因此强调了实现这一目标所需的新步骤，但是整个解决方案是完全可以通过脚本进行的。特定的部分被分组到脚本中，可以运行这些脚本来完成编译流的一部分。这些脚本中使用了显式的名称和路径，但它们当然可以在新设计中进行修改。这些脚本包括:

- `implement_parent_config.tcl`：这将实现顶层静态设计并建立一阶可重构分区（inst_RP），该分区稍后将被细分。
- `subdivide_shifters.tcl`：这将一阶可重构分区细分为两个二级移位函数，每个函数部分可重构。
- `subdivide_counters.tcl`：这将一阶可重构分区细分为两个二级计数函数，每个函数部分可重构。
- `implement_sub_shifters.tcl`：它在 inst_RP 下面的两个二级可重构分区中实现了 shift_right 和 shift_left 可重构模块。
- `implement_sub_counters.tcl`：它在 inst_RP 下面的两个二级可重构分区中实现了 count_up 和 count_down 可重构模块。
- `verify_configurations.tcl`：它在对设计检查点上运行 pr_verify，以确认中包含的可重构模块的兼容性。
- `generate_all_bitstreams.tcl`：它逐个打开检查点，为这个总体设计创建所有可能的部分比特流。
- `run_all.tcl`：这将运行上面的所有脚本，从综合到比特流生成，以编译完整的教程设计。

**3. 综合设计**

`run_synth.tcl` 脚本自动化进行综合阶段。总共调用了七次综合迭代，一次用于静态顶层设计，两次用于一阶可重构模块，四次用于二级可重构模块。

我们打开 Vivado Tcl Shell，定位到 `<Extract_Dir>`，在目录下执行 Tcl 脚本：

```
source run_synth.tcl -notrace 
```

综合完成后，可以在 `Synth` 子目录中的每个命名文件夹下找到每个模块的日志和报告文件，以及最后的检查点。

**4. 集成并实现设计**

当所有检查点都完成综合后，就可以进行设计集成与实现了。接下来将从 Tcl 控制台运行所有的工作流程步骤，也可以在 IDE 中交互式地进行。

**4.1 实现设计流程**

本实验中的步骤由一组 Tcl 脚本管理，这些脚本用于实现嵌套 DFX 设计的每个配置的命令。在运行之前检查每个脚本，看看每个脚本的作用。大多数命令（link_design、route_design、pr_verify、write_bitstream 等）看起来很熟悉，而其他命令（pr_subdivide、pr_recombine）则是新的。关键的细节是它们运行的顺序，因为后面的脚本依赖于前面的脚本。

脚本完成后，打开检查点（例如 `top_route_design.dcp` 或 `top_count_up_route_design.dcp`）来检查结果，注意 DFX 属性和用于创建它们的命令的含义。

第一个实现设计运行建立静态设计和一阶可重构分区。在这一点上，流程与标准 DFX 设计流程没有什么不同，inst_RP 是设计中的唯一可重构分区，位于该级别以下的移位模块与其余的 inst_RP 逻辑一起实现，二级可重构分区还不存在。

首先激活运行脚本来实现父配置：

```
source implement_parent_config.tcl -notrace
```

生成的检查点（`top_route_design.dcp`）是具有单个可重构分区的完整设计映像。此时还没有额外的 DFX 步骤。此检查点将仅用于建立锁定的静态设计映像，这对于随后的所有设计迭代都是通用的。

![](1.png)

打开 `implementation/top_static/top_route_design.dcp`，可以看到这是一个标准的 DFX 设计。inst_RP 具有 HD.RECONFIGURABLE 属性和一个可重构分区的关联 Pblock。

![](2.png)

然后通过下面的脚本创建二级可重构分区：

```
source subdivide_shifters.tcl
```

该脚本将 inst_rp 模块细分为二级可重构分区。pr_subdivide 命令删除 inst_RP 的 HD.RECONFIGURABLE 属性，并将其应用于 inst_shift_upper 和 inst_shift_lower。inst_RP会被 HD.RECONFIGURABLE_CONTAINER 属性标记，表示其曾经是一个可重构分区。通过查看 `top_static_shifters.dcp` 检查点可以看到这一点。

![](3.png)

![](4.png)

可以通过检查 inst_RP 分层实例上的属性直接查询 HD.RECONFIGURABLE_CONTAINER 属性。下面的Tcl命令将返回值 1。

```
get_property HD.RECONFIGURABLE_CONTAINER [get_cells inst_RP]
```

然后在可重构分区中实现二级移位子模块：

```
source implement_sub_shifters.tcl -notrace
```

这将通过两个实现工作流程在二级可重构分区中布局布线 shift_right 和 shift_left 函数。这里使用的命令与标准 DFX 流程相同，但有一点不同的是，第一个配置的起始点包括锁定的顶级静态设计。实现将 inst_RP 级别（reconfig_shifters）的逻辑设计视为静态的，这是在第一次配置完成后由 lock_design -level routing 命令锁定的层次结构级别。

![](5.png)

该脚本以调用 pr_recombine 结束，以创建 shift_right 与 shift_right 组合的布线设计检查点，将 HD.RECONFIGURABLE 属性移回 inst_RP 级别。检查 `top_shift_right_right_recombined.dcp` 的层次结构，可以看到这个属性已经返回到 inst_RP 实例。

![](6.png)

接着用这个脚本创建另一组二阶可重构分区：

```
source subdivide_counters.tcl
```

就像第一个可重构区域细分脚本一样，这个脚本从初始配置（`top_route_design.dcp`）开始，并细分 inst_RP 级别，但这次将分为两个计数器函数。此设计版本的顶级静态与用于移位器的版本相同。

![](7.png)

最后在二级可重构分区中实现计数器子模块：

```
source implement_sub_counters.tcl -notrace
```

同样，与移位器流程一样，使用标准 DFX 工作流程处理每个二阶可重构分区中的两个计数器模块（ count_up 与 count_down）。与移位器一样，重新组合的设计检查点是从第一次通过二级实现流程创建的。

![](8.png)

**4.2 更新静态设计**

就像标准 DFX 设计流程一样，实现结果是在上下文中由上而下创建的。如果在任何时候被认为是静态的设计的任何部分必须更新，则必须重新实现静态以下的可重构模块的所有结果，以确保所有内容保持同步。

例如，如果顶层静态的设计发生了变化，那么所有现有的结果都必须被认为是过时的，并且必须重新编译所有内容。如果对一阶可重构模块（reconfig_shifters 或 reconfig_counters）中的一个进行了更新，则必须重新编译依赖于修改模块的所有结果。可以单独调用这些脚本中的任何一个来根据需要更新结果。

**4.3 通过验证**

就像标准 DFX 设计流程一样，应该使用 pr_verify 检查嵌套的 DFX 设计映像，以确认所有映像都是同步的。与核心实现工具（opt_design 等）一样，pr_verify 将根据标记为可重构的当前单元对设计进行操作。执行以下脚本验证所有兼容的配置：

```
source verify_configurations.tcl
```

这个脚本比较了三对布线设计，每个检查点都对具有相同静态逻辑的检查点进行两两比较。本节描述所做的比较和将在下一步中创建的兼容的比特流。

对 pr_verify 的第一次调用将比较两个重组的检查点，它们都应该具有相同的静态实现结果 top，只有一个可重构分区 inst_RP。这些检查点表示没有嵌套的标准 DFX 设计，即使每个检查点都可以接收适当的二级部分比特流。如果使用二级模块（例如 shift_left 和 count_down）创建其他检查点，然后重新组合，则可以通过 pr_verify 对它们进行比较，并将其 inst_RP 部分比特流添加到此兼容性列表中。对于 inst_RP 的任何其他可重构模块也是如此，即使没有任何细分的二级可重构分区。

对 pr_verify 的第二次调用比较了 shift_right 和 shift_left 二级检查点。这些静态锁定在上层和下层子模块上，因此比较的是顶层和 reconfig_shifters 层次结构的静态逻辑。

与第二次调用非常相似，第三次调用 pr_verify 比较 count_up 和 count down 二级检查点。它们对 top 和 reconfig_counters 进行了静态锁定，因此比较的是上下可重构分区之间的静态逻辑。

**4.4 比特流创建**

实现工具遵循基于当前定义为可重构的单元的标准 DFX 设计规则，这也适用于 write_bitstream。部分比特流将仅为当前持有 HD.RECONFIGURABLE 属性的单元创建。

在 Vivado 中打开任何完全布线的设计检查点，使用 write_bitstream 来生成完整和部分比特流。默认情况下，该命令将为整个设备生成一个标准的完整比特流，并为当前定义为可重构分区的每个单元生成一个部分比特流，两个选项可以将结果限制为其中一个：

- `-cell` 选项将只生成所请求单元的部分比特流。
- `-no_partial_bitfile` 选项将只生成一个标准的完整设备比特流。

运行以下脚本，为已实现的现有配置创建完整和部分比特流集合。为了节省时间和空间，只创建一个完整的设备位流。

```
source generate_all_bitstreams.tcl
```

该脚本逐个打开每个检查点，并写入特定的完整或部分比特流，比特流根据兼容性放置在子文件夹中。每个比特流都是用 `-no_partial_bitfile` 选项或 `-cell` 选项创建的。
