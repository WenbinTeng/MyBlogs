---
title: fpga-nested-dfx
date: 2024-05-29 09:10:29
tags:
typora-root-url: fpga-dfx-nested-flow
---

# FPGA DFX Nested Flow

嵌套 DFX 允许在一个动态重构分区（Reconfigurable Partition，RP）内嵌套一个或多个区域，通过细分器件实现更精细化的重构。该特性允许将 RP 划分为多个子区域，每个子区域均可独立进行部分重构。

目前嵌套 DFX 尚不支持项目模式，因此本实验采用 Tcl 脚本流程作为演示示例。



### 1. 准备设计文件

从 Xilinx 官网下载官方 DFX 教程设计文件：

[下载链接](https://www.xilinx.com/member/forms/download/design-license.html?cid=03e48cb4-ba89-496d-a3d7-cbaa2302ef79&filename=ug947-vivado-partial-reconfiguration-tutorial.zip)

下载完成后，将文件解压至任意具备写权限的本地路径，然后进入解压后的 `nested_dfx` 子目录。本实验中的所有步骤都将在该目录下完成。



### 2. 查看脚本功能

我们查看设计目录中提供的脚本文件，在根目录下有两种类型的脚本。

##### 综合脚本

在根目录下，包含三个在综合流程中会使用的脚本：

- `run_synth.tcl`：用于控制在综合流程中哪些模块会被综合。
- `design_settings.tcl`：用于设置目标开发板参数，并为项目设置相对路径。
- `advanced_settings.tcl` 包含 DFX 工作流程参数的高级设置文件，一般不需要进行修改。

##### 功能脚本

在根目录下，包含用于实现嵌套 DFX 功能的脚本：

- `implement_parent_config.tcl`：用于实现顶层静态设计并建立第一级 RP。
- `subdivide_shifters.tcl`：用于将第一级 RP 细分为用于实现移位功能的两个第二级 RP。
- `subdivide_counters.tcl`：用于将第一级 RP 细分为用于实现计数功能的两个第二级 RP。
- `implement_sub_shifters.tcl`：用于实现第二级 RP 中的移位可重构模块（Reconfigurable Module，RM）。
- `implement_sub_counters.tcl`：用于实现第二级 RP 中的计数 RM。
- `verify_configurations.tcl`：用于对生成的设计检查点（Design Check Point，DCP）执行检查，以确保动态重构的兼容性。
- `generate_all_bitstreams.tcl`：用于生成项目中的所有比特流。

##### 流程脚本

在根目录下，包含一个用于执行整个嵌套 DFX 编译流程的脚本：

- `run_all.tcl`：按照顺序执行项目中的脚本，以自动进行综合-实现-生成比特流的流程。



### 3. 综合设计

`run_synth.tcl` 脚本用于自动化进行综合阶段，其将会执行七次综合步骤，一次用于顶层设计，两次用于第一级 RM，四次用于第二级 RM。具体使用步骤如下：

1. 启动 Vivado Tcl Shell。

2. 将当前的工作目录切换至 `nested_dfx` 。

3. 确认 `run_synth.tcl` 中的 `xboard` 变量已经被修改为目标开发板型号。

4. 在 Tcl 终端中执行以下命令以运行综合脚本：

   ```tcl
   source run_synth.tcl -notrace
   ```

   运行结束后，你可以在目录 `Synth/` 目录下查看所有模块的综合结果、日志于报告文件。主要的日志包括：

   - `run.log`：记录综合阶段的运行汇总信息；
   - `command.log`：打印脚本执行的完整 Tcl 命令序列；
   - `critical.log`：汇总综合过程中出现的所有关键警告（Critical Warnings）。



### 4. 集成设计并实现设计

在所有的模块完成综合并生成相应的 DCP 后，即可利用这些检查点组合进行设计集成。这里我们将在 Tcl 终端中运行脚本以执行这一流程，你也可以在 Vivado IDE 的 GUI 中进行实现相关的操作。

##### 4.1 集成与实现

1. 执行第一个脚本，进行顶层设计的集成与实现：

   ```tcl
   source implement_parent_config.tcl -notrace
   ```

   `implement_parent_config.tcl` 脚本已经提供了集成与实现顶层设计的执行流程。顶层设计包括了静态设计和一个第一级 RP。这里，执行流程与标准 DFX 的执行流程一致。具体的执行流程大致为：

   - 加载顶层设计 DCP 与板卡对应的约束文件；
   - 加载 RM 综合生成的 DCP 与包含布局规划的约束文件；
   - 使用 `link_design` 命令链接整个设计；
   - 保存集成结果至 DCP 文件；
   - 优化、布局和布线当前设计；
   - 保存布线结果至 DCP 文件。

   执行脚本后生成的 DCP 状态如下图所示：

   <img src="1.png" style="zoom:80%;" />

   打开已写入 `Implement/top_static` 文件夹的 `top_route_design.dcp` 文件，可以看到这是一个标准的 DFX 设计。第一级 RP `inst_RP` 具有 `HD.RECONFIGURABLE` 属性，并关联了一个 Pblock。

   <img src="2.png" style="zoom:80%;" />

2. 执行第二个脚本，以创建第二级 RP：

   ```tcl
   source subdivide_shifters.tcl
   ```

   `subdivide_shifters.tcl` 脚本会将设计中的第一级 RP 进一步划分为两个第二级 RP。其中使用的 `pr_subdivide` 命令将为 `inst_RP` 移除 `HD.RECONFIGURABLE` 属性，并将其添加到两个第二级 RP 中，即 `inst_shift_upper` 和 `inst_shift_lower`。接着，其会为 `inst_RP` 添加 `HD.RECONFIGURABLE_CONTAINER` 属性，表明它在曾经是一个 RP。在生成的检查点 `top_static_shifters.dcp` 中可以看到上述变化。

   <img src="3.png" style="zoom:80%;" />

   <img src="4.png" style="zoom:80%;" />

   `HD.RECONFIGURABLE_CONTAINER` 属性也可以通过检查 `inst_RP` 层次实例上的属性直接查询。使用以下 Tcl 命令进行查询的返回值将为 1。

   ```tcl
   get_property HD.RECONFIGURABLE_CONTAINER [get_cells inst_RP]
   ```

3. 执行第三个脚本，以实现两个移位器的 RM：

   ```tcl
   source implement_sub_shifters.tcl -notrace
   ```

   `implement_sub_shifters.tcl` 脚本将会使用两种方法分别实现第二级 RP 中的 `shift_right` 和 `shift_left` 模块：

   -  `shift_right` 模块的实现将使用仅包含初始静态设计的 DCP 构成的上下文作为起点。当前的第一级 RP 处于已布线但未锁定的状态；
   - `shift_left` 模块的实现将使用包含静态设计与 RP 容器静态部分的 DCP 构成的上下文作为起点，当前的第一级 RP （容器）处于锁定的状态，是使用  `lock_design -level routing` 命令进行锁定后的结果。

   <img src="5.png" style="zoom:80%;" />

   两个模块实现完成之后，脚本将使用 `pr_recombine` 命令创建由两个 `shift_right` 模块填充两个第二级 RP 的 DCP，并将 `HD.RECONFIGURABLE` 属性移回 `inst_RP` 层级。可以通过检查 `top_shift_right_right_recombined.dcp` 的层次结构，看到属性已返回到 `inst_RP` 实例。

   <img src="6.png" style="zoom:80%;" />

4. 执行第四个脚本，以创建新的第二级 RP：

   ```tcl
   source subdivide_counters.tcl
   ```

   与第二个脚本类似，`subdivide_counters.tcl` 脚本会将设计中的第一级 RP 进一步重新划分为两个第二级 RP，用于实现计数器功能。这个设计中包含的顶层设计与第一个设计完全相同。

   <img src="7.png" style="zoom:80%;" />

5. 执行第五个脚本，以实现两个计数器的 RM：

   ```tcl
   source implement_sub_counters.tcl -notrace
   ```

   与第三个脚本的实现流程类似，`implement_sub_counters.tcl` 脚本使用标准的 DFX 流程实现第二级 RP 中的所有模块，在脚本最后使用 `pr_recombine` 命令重新组合并生成 DCP。

   <img src="8.png" style="zoom:80%;" />

##### 4.2 静态设计更新

与标准的 DFX 设计流程一致，RM 的实现结果是根据静态设计构成的上下文中生成的。如果静态设计中的内容发生改变，那么该静态部分下的所有 RM 的必须重新实现，以确保所有内容保持同步。

例如，如果顶层设计的静态部分有设计变更，则所有现有结果都应视为过时，必须重新编译所有内容。如果对一级可重构模块（如 `reconfig_shifters` 或 `reconfig_counters`）进行了更新，则依赖于修改模块的所有结果都需要重新编译。可以单独调用这些脚本中的任何一个，根据需要更新结果。

##### 4.3 验证结果

与标准的 DFX 设计流程一致，应使用 PR Verify 检查嵌套 DFX 设计生成的结果，以确认所有的结果保持同步。Vivado 的 `pr_verify` 命令将根据当前标记为可重构的单元对设计进行操作。鉴于此，应在相同的当前静态设计存在情况下进行同类比较。通过运行此脚本验证所有兼容配置：

```tcl
source verify_configurations.tcl
```

`verify_configurations.tcl` 脚本通过调用三次 `pr_verify` 比较了三对已经布线的设计，其内容如下：

- `top_shift_right_right_recombined.dcp` vs `top_count_up_up_recombined.dcp`：这将比较两个重组后的 DCP，应该拥有完全相同的静态部分，并且包含一个 `inst_RP` RP。这两个 DCP 是标准的 DFX 设计，不包含任何嵌套结构。
- `top_shift_right_right_route_design.dcp` vs `top_shift_left_left_route_design.dcp`：这将比较 `shift_right` 和 `shift_left` 的第二级检查点，由于这两个 DCP 的静态逻辑已锁定到上下子模块，因此比较的是顶层和 `reconfig_shifters` 层次结构之间的静态逻辑。
- `top_count_up_up_route_design.dcp` vs `top_count_down_down_route_design.dcp`：与上一个比较类似，这将比较 `count_up` 和 `count_down` 的第二级检查点。



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
