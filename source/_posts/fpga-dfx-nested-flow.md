---
title: fpga-nested-dfx
date: 2024-05-29 09:10:29
tags:
typora-root-url: fpga-dfx-nested-flow
---

# FPGA DFX Nested Flow

Vivado 的嵌套 DFX （Nested DFX）功能允许在一个动态重构分区（Reconfigurable Partition，RP）内嵌套一个或多个区域，通过细分器件实现更精细化的重构。该特性允许将 RP 划分为多个子区域，每个子区域均可独立进行部分重构。

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

在所有的模块完成综合并生成相应的设计检查点（Design Check Point，DCP）之后，就可以利用这些 DCP 组合链接生成集成的设计。这里我们将通过 Tcl 脚本依次完成顶层设计集成、嵌套 RP 拆分、子模块实现与结果验证等步骤。

##### 4.1 集成与实现

1. 执行第一个脚本，进行顶层设计的集成与实现：

   ```tcl
   source implement_parent_config.tcl -notrace
   ```

   `implement_parent_config.tcl` 脚本提供了针对顶层设计的完整集成与实现流程。顶层设计包括静态设计和一个第一级 RP。这里，执行流程与标准 DFX 的执行流程一致。具体的执行流程大致为：

   - 加载综合顶层设计的 DCP，以及板卡级约束文件；
   - 加载综合可重构模块（Reconfigurable Module，RM） 的 DCP，以及用于模块布局规划的约束文件；
   - 使用 `link_design` 命令链接整个设计结构；
   - 保存集成结果至 DCP 文件；
   - 优化、布局和布线当前设计；
   - 保存布线结果至 DCP 文件。

   执行脚本后生成的 DCP 状态如下图所示：

   <img src="1.png" style="zoom:80%;" />

   打开已写入 `Implement/top_static` 文件夹的 `top_route_design.dcp` 文件，可以看到这是一个标准的 DFX 设计层次结构。第一级 RP `inst_RP` 具有 `HD.RECONFIGURABLE` 属性，并绑定了一个 Pblock。

   <img src="2.png" style="zoom:80%;" />

2. 执行第二个脚本，以创建第二级 RP：

   ```tcl
   source subdivide_shifters.tcl
   ```

   `subdivide_shifters.tcl` 脚本会将设计中的第一级 RP 进一步划分为两个第二级 RP，以实现不同的移位功能。该脚本会使用 `pr_subdivide` 命令将为 `inst_RP` 移除 `HD.RECONFIGURABLE` 属性，并将其添加到两个第二级 RP 中，即 `inst_shift_upper` 和 `inst_shift_lower`。接着，该脚本会为 `inst_RP` 添加 `HD.RECONFIGURABLE_CONTAINER` 属性，表明它在曾经是一个 RP，现在已经是下一级 RP 的容器。查看生成的检查点 `top_static_shifters.dcp` 中可以看到上述变化。

   <img src="3.png" style="zoom:80%;" />

   <img src="4.png" style="zoom:80%;" />

   也可以通过检查 `inst_RP` 层次结构实例上的属性，查询 `HD.RECONFIGURABLE_CONTAINER` 属性的值。也可以使用以下命令进行查询，这里的返回值将为 1：

      ```tcl
      get_property HD.RECONFIGURABLE_CONTAINER [get_cells inst_RP]
      ```

3. 执行第三个脚本，以实现两个移位器的 RM：

   ```tcl
   source implement_sub_shifters.tcl -notrace
   ```

   `implement_sub_shifters.tcl` 脚本将会使用两种方法分别实现第二级 RP 中的 `shift_right` 和 `shift_left` 模块：

   - `shift_right` 模块实现使用的上下文为：锁定的静态设计部分、布线的第一级 RP。
   - `shift_left` 模块实现使用的上下文为：锁定的静态设计部分、锁定的第一级 RP。

   <img src="5.png" style="zoom:80%;" />

   这两种方法都能让模块正确地实现。个人感觉这两种流程在生成的结果上没有区别，只是第一次实现两个第二级 RM 时，还无法确切地知道第二级 RM 的接口在第一级 RP 中的布局布线（可以类比于第一级 RM在顶层设计中实现的过程，锁定的顶层设计静态部分 DCP 是在第一个 RM 实现之后才生成的。因此，这是一个嵌套的过程。）。为第一级 RP 生成锁定的 DCP 能为第二级 RM 的实现增强隔离性，减少布线错误。

   两个模块实现完成之后，脚本将使用 `pr_recombine` 命令，将两个 `shift_right` RM 填充到两个第二级 RP 中，并生成 DCP。同时，这个命令会将 `HD.RECONFIGURABLE` 属性移回 `inst_RP` 层级。可以通过检查 `top_shift_right_right_recombined.dcp` 的层次结构，看到该属性已返回到 `inst_RP` 实例。

   <img src="6.png" style="zoom:80%;" />

4. 执行第四个脚本，以创建新的第二级 RP：

   ```tcl
   source subdivide_counters.tcl
   ```

   与第二个脚本类似，`subdivide_counters.tcl` 脚本会将设计中的第一级 RP 进一步重新划分为两个第二级 RP，以实现不同的计数功能。这个设计中包含的顶层设计的静态部分与第一个设计完全相同。

   <img src="7.png" style="zoom:80%;" />

5. 执行第五个脚本，以实现两个计数器的 RM：

   ```tcl
   source implement_sub_counters.tcl -notrace
   ```

   与第三个脚本的实现流程类似，`implement_sub_counters.tcl` 脚本使用标准的 DFX 流程实现第二级 RP 中的所有模块，在脚本最后使用 `pr_recombine` 命令重新组合并生成 DCP。

   <img src="8.png" style="zoom:80%;" />

##### 4.2 静态设计更新依赖说明

如同标准 DFX 流程，RM 的实现必须基于静态设计的上下文。如果顶层静态逻辑发生修改，所有相关 RM 必须重新编译，以保持上下文一致性。

例如，如果顶层设计的静态部分有设计变更，则所有现有结果都应视为过时，必须重新编译所有内容。如果对第一级可重构模块（如 `reconfig_shifters` 或 `reconfig_counters`）进行了更新，则依赖于该第一级模块的所有结果都需要重新编译。可以重新调用前述的脚本，根据需要更新结果。

##### 4.3 验证结果

与标准的 DFX 设计流程一致，应使用 PR Verify 检查嵌套 DFX 设计生成的结果，以确认所有结果的兼容性。Vivado 的 `pr_verify` 命令将根据当前标记为可重构的单元（Cell）对设计进行检查。因此，应该在相同的静态设计下进行同级别的模块进行比较。以下脚本可以验证本例中的所有配置：

```tcl
source verify_configurations.tcl
```

`verify_configurations.tcl` 脚本通过调用三次 `pr_verify` 比较三对已经布线的设计，其内容如下：

- `top_shift_right_right_recombined.dcp` vs `top_count_up_up_recombined.dcp`：比较两个重组后的 DCP，它们的静态部分应该相同，并包含一个 `inst_RP` RP。
- `top_shift_right_right_route_design.dcp` vs `top_shift_left_left_route_design.dcp`：比较两个移位子模块（`shift_right` 和 `shift_left` ）之间的静态一致性。这里将会比较锁定的顶层设计静态部分、锁定的第一级 `reconfig_shifters` RM 之中的静态逻辑是否兼容。
- `top_count_up_up_route_design.dcp` vs `top_count_down_down_route_design.dcp`：与上一个比较类似，这里将比较两个计数子模块（ `count_up` 和 `count_down` ）之间的静态一致性。

##### 4.4 生成比特流

在嵌套 DFX 流程中，Vivado 将为 DCP 中具有 `HD.RECONFIGURABLE` 属性的模块生成部分比特流。请注意，默认情况下使用  `write_bitstream` 命令会为整个设备生成标准完整比特流，并为当前定义为 RP 的每个区域生成部分比特流。以下两个选项可以选择只生成一种比特流：

- `-cell` 选项将仅为指定的单元生成部分比特流。
- `no_partial_bitfile` 选项将仅会生成设备的完整比特流。

运行以下脚本可以生成所有配置的完整比特流与部分比特流：

```tcl
source generate_all_bitstreams.tcl
```

`generate_all_bitstreams.tcl` 脚本将会逐个打开实现的 DCP 文件，使用 `-no_partial_bitfile` 选项（下面列出的第一个比特流）或 `-cell` 选项（其他所有比特流）创建比特流。生成的比特流根据兼容性被放到不同的文件夹中，如下所示：

- `Bitstreams`
  - `top_shift_right_right.bit`
- `Bitstreams/inst_RP`
  - `inst_RP_shift_right_right_recombined_partial.bit`
  - `inst_RP_count_up_up_recombined_partial.bit`
- `Bitstreams/inst_shift`
  - `shift_right_upper_partial.bit`
  - `shift_right_lower_partial.bit`
  - `shift_left_upper_partial.bit`
  - `shift_left_lower_partial.bit`
- `Bitstreams/inst_count`
  - `count_up_upper_partial.bit`
  - `count_up_lower_partial.bit`
  - `count_down_upper_partial.bit`
  - `count_down_lower_partial.bit`

如果你使用的是 UltraScale 设备，那么 Vivado 还会为你创建清除比特流文件，用于烧写部分比特流之前清除 RP 的内容。上述的每一个部分比特流文件对应一个清除比特流文件，以 `_clear` 结尾。

另外，脚本中还提供了创建灰盒（Grey Box）的功能，用于覆盖 RP 内的功能，并向外输出恒定值。如果需要为每个 RP 生成对应的灰盒模块，那么可以在脚本中将 `grey` 变量设置为 `true`。

`generate_all_bitstreams.tcl` 脚本会为本例嵌套 DFX 流程中创建的不同级别的 RP 生成对应的灰盒，具体的文件层级结构如下：

- `Bitstreams/inst_RP`
  - `inst_RP_grey_partial.bit`
  - `reconfig_shifters_grey_grey_partial.bit`
  - `reconfig_counters_grey_grey_partial.bit`
- `Bitstreams/inst_shift`
  - `shift_upper_grey_partial.bit`
  - `shift_lower_grey_partial.bit`
- `Bitstreams/inst_count`
  - `count_upper_grey_partial.bit`
  - `count_lower_grey_partial.bit`

总之，采用自上而下的方式构建设计结果，使用 `pr_subdivide` 命令锁定每个相对静态层。随后，若要返回更高层次的可重配置分区，则使用 `pr_recombine` 创建检查点，以便在该层级生成部分比特流。



### 5. 上板测试

在完成所有配置的完整与部分比特流生成后，我们可以将其部署至 FPGA 开发板进行功能验证。

##### 烧写完整配置

现在我们将完整比特流烧写到 FPGA 开发板上。

1. 将开发板连接到计算机并通电。
2. 在 Vivado IDE 中，选择 *Flow > Open Hardware Manager*。
3. 点击绿色横幅上的 *Open target*，按照向导步骤与 FPGA 开发板建立通信连接。
4. 右键单击目标器件，选择 *Program Device*。
5. 定位至 `Bitstreams` 文件夹选择 `top_shift_right_right.bit` 文件，点击 *Program* 按钮对设备进行烧写。

现在，你可以看到当前的两组 GPIO LED 正在执行两个相同的任务：两组四个 LED 正在向右移位。

你可以留意配置整个设备所需的时间，与后面部分重构设备花费的时间进行比较。

当前的设备上，包含顶层的静态设计、第一级 `reconfig_shifters` RM、第二级 `shift_right` 与 `shift_right` RM。

##### 烧写部分配置

特别地，对于 UltraScale 器件，在加载任何部分比特流之前，必须先加载对应的清除比特流，详细的操作请参考官方文档。这里我们使用 UltraScale+ 器件作为实例，以下是具体步骤：

1. 在绿色横幅上选择 *Program device*。定位至 `Bitstreams/inst_shift` 文件夹，选择 `shift_left_upper_partial.bit` 文件，然后点击 *Program* 以烧写设备。此时上部移位部分 RP 已改为左移模块，而下部 RP 仍保持右移。DONE 信号也恢复为高电平（on）。

   如果要切换至计数功能，则必须先烧写第一级 `reconfig_counters` RM。如果此时直接烧写第二级 `count_up` 或者 `count_down` RM 将无法正常工作，因为这些模块无法正确地与顶层设计的静态部分建立连接。

2. 点击绿色横幅上的 *Program device*，定位至 `Bitstreams/inst_RP` 文件夹，选择 `inst_RP_count_up_up_recombined_partial.bit` 文件，然后点击 *Program* 以烧写设备。此时两组 LED 开始向上计数。

   此时，第一级 RP 的功能已经成功替换，可以对第二级的 RP 进行烧写。

3. 点击绿色横幅上的 *Program device*。定位至 `Bitstreams/inst_count` 文件夹，选择 `count_down_lower_partial.bit` 文件，点击 *OK* 对设备进行烧写。此时上部移位部分 RP 仍在向上计数，而下部 RP 已转为向下计数。

至此，针对 UltraScale+ 器件的配置流程已完成。



### 6. 命令说明

##### pr_subdivide

该命令用于将一个 RP 分解为一个或多个更低级别的 RP。

```
pr_subdivide
Description: Subdivide an RP into one or more lower-level 
RPs when using the Nested Dynamic Function eXchange solution.
Syntax: 
pr_subdivide [-cell <arg>] [-subcells <arg>] [-quiet] [-verbose] [<from_dcp>]
Usage: 
Name     Description
-------------------------
[-cell]    (Required) Specify parent RP module name
[-subcells]  (Required) Specify child RP module names
[-quiet]   Ignore command errors
[-verbose]  Suspend message limits during command execution
[<from_dcp>] (Required) Specify OOC synthesized checkpoint path for the RM specified by option -cell
```

在 Vivado 中打开一个完全布线的初始设计检查点后，运行 pr_subdivide 将自动执行以下任务：

- 执行 `lock_design -level routing` 命令将 RP 之外的静态逻辑锁定；
- 执行 `update_design -black_box` 命令将 RP 替换为黑盒；
- 加载一个 RP 对应的 RM 的综合后的 DCP，即命令中的 `<from_dcp>` 标识。该 RM 必须包含一个或多个层次结构实例。
- 将 `HD.RECONFIGURABLE` 属性从初始分区（由 `-cell` 选项指定）转移到至一个或多个下级分区（由 `-subcells` 选项指定）。
- 在初始分区上放置 `HD.RECONFIGURABLE_CONTAINER` 属性作为占位符，以供后续的 `pr_recombine` 命令使用。

##### pr_recombine

该命令用于移除所有较低级别的 RP，将 RP 定义恢复到父单元。

```
pr_recombine
Description: Re-establish a parent cell as a RP while removing 
lower-level RPs when using the Nested Dynamic Function 
eXchange solution.
Syntax: 
pr_recombine [-cell <arg>] [-quiet] [-verbose]
Usage: 
Name    Description
-----------------------
[-cell]   (Required) Specify reconfigurable container module name
[-quiet]  Ignore command errors
[-verbose] Suspend message limits during command execution
```

`pr_recombine` 将 `HD.RECONFIGURABLE` 属性移动到目标单元，并从其下方的子单元中移除该属性。指定的单元必须拥有由 `pr_subdivide` 设置的 `HD.RECONFIGURABLE_CONTAINER` 属性，以标识其为有效目标。

