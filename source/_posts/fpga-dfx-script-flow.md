---
title: fpga-dfx-script-flow
date: 2024-05-29 21:22:14
tags:
typora-root-url: fpga-dfx-script-flow
---
# FPGA DFX Script Flow

本实验介绍了动态功能交换（DFX）基本流程。首先，您将使用脚本单独综合静态模块和每个可重构模块的变体。然后在 IDE 中，您将使用 Pblocks 约束可重构模块的位置，并实现该设计的初始配置。接着，您将通过锁定设计的静态部分、使用变体更新可重构模块以及重新运行实现来实现可选配置。最后，您将验证每个实现的可重构模块是否与设计的静态部分兼容，如果兼容，则生成比特流。

**1. 准备设计文件**

下载 Xilinx 提供的[参考设计文件](https://www.xilinx.com/member/forms/download/design-license.html?cid=03e48cb4-ba89-496d-a3d7-cbaa2302ef79&filename=ug947-vivado-partial-reconfiguration-tutorial.zip)，解压后进入 `led_shift_count_7s/` 文件夹，假设路径为 `<Extract_Dir>`。

**2. 阐述脚本功能**

查看设计文档中提供的脚本，文件 `run_dfx.tcl` 和 `advanced_settings.tcl` 位于根目录。`run_dfx.tcl` 脚本包含运行 DFX 所需的最小设置。`advanced_settings.tcl` 包含默认的工作流程设置，只能由有经验的用户修改。

首先介绍主要脚本，在 `led_shift_count_7s/` 中，在文本编辑器中打开 `run_dfx.tcl`。这是主脚本，您可以在其中定义设计参数、设计源文件和设计结构，这是编译完整的 DFX 设计所需修改的唯一文件。在位于 `Tcl_HD` 目录中的 `README.txt` 中查找有关 `run_dfx.tcl`、`advanced_settings.tcl` 和底层脚本的更多详细信息。以下是 `run_dfx.tcl` 的更多细节：

- 在“Define target demo board”下，您可以选择支持此设计的众多演示板卡之一。
- 在“flow control”下，您可以控制运行合成和实现的哪些阶段。在本教程中，只有综合是由脚本运行的，而实现、验证和比特流生成是交互运行的。如果需要通过脚本运行这些附加步骤，请将工作流变量（例如 run. primpl）设置为1。
- “Output Directories”和“Input Directories”设置了设计源文件和结果文件所需的文件结构。您必须在这里反映对文件结构的任何更改。
- “Top Definition”和“RP Module Definitions”允许您引用设计中的所有源文件。“Top Definition”部分涵盖了静态设计所需的所有资源，包括约束和 IP。“RP Module Definitions”部分对可重构分区做了相同的操作，识别每个可重构分区并列出每个可重构分区的所有可重构模块变体。
  - 本教程的设计有两个可重构分区（inst_shift 和 inst_count），每个可重构分区有两个可重构模块变体。
- “Configuration Definition”部分定义了组成配置的静态和可重构模块集。
  - 本教程的设计在主脚本中定义了两个配置：config_shift_right_count_up_implementation 和 config_shift_left_count_down_import。
  - 您可以通过添加可重构模块或组合现有可重构模块来创建更多配置。

接着介绍支持脚本。在 `Tcl_HD` 子目录下，存在几个 Tcl 支持脚本。这些脚本由 `run_dfx.tcl` 调用，它们管理 DFX 工作流程的特定细节。下面提供了一些关于几个关键 DFX 脚本的细节：

- `step.tcl` - 通过监视检查点来管理设计的当前状态。
- `synthesize.tcl` - 管理有关综合阶段的所有细节。
- `implement.tcl` - 管理有关模块实现阶段的所有细节。
- `dfx_utils.tcl` - 管理有关 DFX 设计顶层实现的所有细节。
- `run.tcl` - 为综合和实现启动实际运行。
- `log_utils.tcl` - 处理流程中关键点上的报告文件创建

**3. 综合设计**

`run_dfx.tcl` 脚本自动化了本教程的综合阶段。将调用五次综合迭代，一次用于静态顶层设计，四次用于四个可重构模块中的每个模块。

我们打开 Vivado Tcl Shell，定位到 `<Extract_Dir>`，在目录下执行 Tcl 脚本：

```
source run_dfx.tcl -notrace
```

综合完成后，可以在 `Synth` 子目录中的每个命名文件夹下找到每个模块的日志和报告文件，以及最后的检查点。

**4. 集成并实现设计**

当所有检查点都完成综合后，就可以进行设计集成与实现了。接下来将从 Tcl 控制台运行所有的工作流程步骤，也可以在 IDE 中交互式地进行。

打开 Vivado IDE，定位到 `<Extract_Dir>`。从以下命令中选择芯片和板卡进行设置：

```
set part "xc7k325t-ffg900-2"
set board "kc705"

set part "xc7vx485t-ffg1761-2"
set board "vc707"

set part "xc7vx690t-ffg1761-2"
set board "vc709"

set part "xc7a200t-fbg676-2"
set board "ac701"
```

通过在 Tcl 控制台中发出以下命令来在内存中创建设计：

```
create_project -in_memory -part $part
```

通过以下命令加载静态设计：

```
add_files ./Synth/Static/top_synth.dcp
```

通过以下命令来加载顶层设计约束：

```
add_files ./Sources/xdc/top_io_$board.xdc
set_property USED_IN {implementation} [get_files ./Sources/xdc/top_io_$board.xdc]
```

通过以下命令来加载 shift 和 count 函数的前两个综合检查点：

```
add_files ./Synth/shift_right/shift_synth.dcp
set_property SCOPED_TO_CELLS {inst_shift} [get_files ./Synth/shift_right/shift_synth.dcp]
add_files ./Synth/count_up/count_synth.dcp
set_property SCOPED_TO_CELLS {inst_count} [get_files ./Synth/count_up/count_synth.dcp]
```

使用 link_design 命令将整个设计链接在一起：

```
link_design -mode default -reconfig_partitions {inst_shift inst_count} -part $part -top top
```

保存这个初始配置的集成设计状态：

```
write_checkpoint -force ./Checkpoint/top_link_right_up.dcp
```

**5. 建立设计布局规划**

接下来，将创建一个布局规划来定义部分重构的区域。

首先，在 Netlist 窗口中选择 inst_count 实例。右键单击，选择 Floorplanning > Draw Pblock ，或选择 Draw Pblock 工具栏按钮，在 X0Y3 时钟区域的左侧绘制一个高窄框。在这一点上，确切的大小和形状并不重要，但保持盒子在时钟区域内。

![](1.png)

虽然这个可重构模块只需要 CLB 资源，但是绘制的 Pblock 还包括 RAMB18、RAMB36 或 DSP48 资源，这是允许的。如果需要，可以使用 Pblock 属性窗口的 General 视图来添加这些属性。Statistics 视图显示当前加载的可重构模块的资源需求。

在 Properties 视图中，选择 RESET_AFTER_RECONFIG 的复选框，以便在重新配置完成后使用该模块中逻辑的专用初始化。

然后对 inst_shift 实例重复上述步骤，只不过这次将 Pblock 绘制在时钟区域 X1Y1 的右侧。此可重构模块包括 Block RAM 实例，因此必须包含资源类型。如果忽略这一需求，则 Statistics 视图中的 RAMB 详细信息将以红色显示。

![](2.png)

然后选择 Reports > Report DRC 运行 DFX 设计规则检查。您可以取消选中 All Rules，然后仅选中 Dynamic Function eXchange，将此报告集中在 DFX DRCs 上。

![](3.png)

目前报告了一个或两个 DRC，有两种解决它们的方法。在本实验中，您将使用一个方法用于 inst_shift，另一个方法用于 inst_count。

第一个 DRC 是一个错误，HDPR-10，报告 RESET_AFTER_RECONFIG 要求 Pblock 帧对齐。要解决第一个 DRC 错误，请确保 Pblock 的高度与时钟区域边界对齐。将 inst_shift 的 Pblock 拉伸顶部和底部边缘以匹配 X1Y1 的时钟区域边界，如下图所示。

![](4.png)

另一个可能的 DRC 是警告，HDPR-26，报告可重构 Pblock 的左或右边缘在不适当的边界上终止。左或右边缘不能分割 INT 列。如果要手动避免此 DRC 警告，需要将 inst_shift 的 Pblock 放大至边缘，以查看违规发生的位置。将 Pblock 的边框向左或向右移动一列，如下图的黄色箭头所示，这样它就会落在两个资源类型（例如 CLB-CLB 或 CLB-RAMB）之间，而不是落在 CLB-INT 或 RAMB-INT 之间。

![](5.png)

再次运行 DRC，以确认已经解决了 inst_shift 实例的错误和警告。

调整可重构 Pblock 的大小和形状的另一种方法是使用 SNAPPING_MODE 特性。此功能自动调整边缘以与合法边界对齐。如果选择 RESET_AFTER_RECONFIG 特性，它将使 Pblock 更高，与时钟区域边界对齐，也使 Pblock 变窄，根据需要调整左边缘和右边缘。注意，如果使用 SNAPPING_MODE 对 Pblock 进行更改，可用资源的数量和类型就会改变。

在 Device 窗口中选择 inst_count 的 Pblock，并在 Pblock Properties 窗口的 Properties 视图中，将 SNAPPING_MODE 的值从 OFF 更改为 ROUTING （或 ON）。

再次运行 DRC，确认所有错误都已解决。可能仍然会报告建议信息，特别是如果 Pblock 位于设备的边缘附近。

使用以下命令保存这些 Pblock 和相关属性：

```
write_xdc ./Sources/xdc/top_all.xdc
```

这将导出设计中的所有当前约束，包括之前从 `top_io_$board.xdc` 导入的约束。这些约束可以在它们自己的 XDC 文件中进行管理，也可以在运行脚本中进行管理（就像 HD.RECONFIGURABLE 所做的那样）。

**6. 实现第一个配置**

接下来，将进行布局布线，并准备设计的静态部分，以便使用新的可重构模块进行重用。

使用以下命令进行优化、布局和布线：

```
opt_design
place_design
route_design
```

运行结束之后，在 Device 视图中检查设计状态，如下图所示。place_design 之后需要注意的一点是引入了 Partition Pins。这些是静态逻辑和可重构逻辑之间的物理接口点。它们是可重构模块的每个 I/O 必须经过的互连块中的锚点。它们在布局的设计视图中显示为白框。

对于 pblock_shift，它们出现在该 Pblock 的顶部，因为到静态区域的连接就在设备上该区域的 Pblock 外部。对于 pblock_count，它们出现在用户定义区域之外，因为 SNAPPING_MODE 垂直收集更多的帧以添加到可重构分区。

![](6.png)

要在 GUI 中找到这些 Partition Pins 是比较容易的，在 Netlist 窗格中选择该可重构模块，在 Cell Properties 属性窗格中选择 Cell Pins 选项卡就可以了。使用以下命令可以选择所有引脚：

```
select_objects [get_pins inst_shift/*]
```

使用布线资源工具栏按钮在抽象布线信息和实际布线信息之间切换，并更改布线资源本身的可见性。在这一视图上，设计中的所有网络都是完全布线的。

![](7.png)

通过以下命令保存完整的设计检查点并创建报告文件：

```
write_checkpoint -force Implement/Config_shift_right_count_up_implement/top_route_design.dcp 

report_utilization -file Implement/Config_shift_right_count_up_implement/top_utilization.rpt

report_timing_summary -file Implement/Config_shift_right_count_up_implement/top_timing_summary.rpt
```

通过以下两个命令为每个可重构模块保存检查点：

```
write_checkpoint -force -cell inst_shift Checkpoint/shift_right_route_design.dcp
write_checkpoint -force -cell inst_count Checkpoint/count_up_route_design.dcp
```

至此，您已经创建了一个完全实现的 DFX 设计，您可以从中生成完整和部分比特流。此配置的静态部分用于所有后续配置。要隔离静态设计，请删除当前的可重构模块。通过发出以下命令清除可重构模块逻辑：

```
update_design -cell inst_shift -black_box
update_design -cell inst_count -black_box
```

发出这些命令会导致设计更改，如下图所示：

![](8.png)

![](9.png)

发出以下命令来锁定所有布局布线：

```
lock_design -level routing
```

因为在 lock_design 命令中没有识别单元，所以内存中的整个设计都会受到影响。所有已经布线的网络现在显示为锁定，如下图中虚线所示。所有布局的组件从蓝色变为橙色，表示它们也被锁定。

![](10.png)

使用以下命令，写出剩余的静态检查点，用于后面的配置：

```
write_checkpoint -force Checkpoint/static_route_design.dcp
```

在进入下一个配置之前关闭此设计：

```
close_project
```

**7. 实现第二个配置**

现在静态设计结果已经建立并锁定，您可以将其用作实现进一步的可重构模块的上下文。

使用以下命令创建一个新的内存中的工程：

```
create_project -in_memory -part $part
```

通过以下命令加载静态设计：

```
add_files ./Checkpoint/static_route_design.dcp
```

通过以下命令来加载 shift 和 count 函数的第二个综合检查点：

```
add_files ./Synth/shift_left/shift_synth.dcp

set_property SCOPED_TO_CELLS {inst_shift} [get_files ./Synth/shift_left/shift_synth.dcp]

add_files ./Synth/count_down/count_synth.dcp

set_property SCOPED_TO_CELLS {inst_count} [get_files ./Synth/count_down/count_synth.dcp]
```

使用 link_design 命令将整个设计链接在一起：

```
link_design -mode default -reconfig_partitions {inst_shift inst_count} -part $part -top top
```

此时，完整的配置已被加载。然而，现在的静态设计是已布线和锁定的，可重构逻辑仍然只是一个网表。这里的布局和布线只适用于可重构模块。

通过以下命令，在静态上下文中优化、布局和布线新的可重构模块：

```
opt_design 
place_design 
route_design
```

现在该设计再次完全实现，现在有了新的可重构模块变体。布线是由虚线（锁定的）和实线（新布线的）组成的混合体，如下图所示。

![](11.png)

通过以下命令保存完整的设计检查点和报告文件：

```
write_checkpoint -force Implement/Config_shift_left_count_down_import/top_route_design.dcp

report_utilization -file Implement/Config_shift_left_count_down_import/top_utilization.rpt

report_timing_summary -file Implement/Config_shift_left_count_down_import/top_timing_summary.rpt
```

通过以下两个命令，为每个可重构模块保存检查点：

```
write_checkpoint -force -cell inst_shift Checkpoint/shift_left_route_design.dcp
write_checkpoint -force -cell inst_count Checkpoint/count_down_route_design.dcp
```

至此已经实现了静态设计和所有可重构模块变体。对于每个可重构分区具有两个以上可重构模块的设计，重复此过程。

**8. 生成比特流**

在生成比特流之前，请对所有的配置进行验证，以确保每个配置的静态部分匹配相同，因此生成的比特流可以安全地在片上使用。PR Verify 功能检查完整的静态设计，包括分区的引脚，直至确认它们是相同的。

在 Tcl 控制台中运行 pr_verify 命令：

```
pr_verify Implement/Config_shift_right_count_up_implement/top_route_design.dcp Implement/Config_shift_left_count_down_import/top_route_design.dcp
```

如果成功，该命令将返回以下消息。

```
INFO: [Vivado 12-3253] PR_VERIFY: check points Implement/Config_shift_right_count_up/
top_route_design.dcp and Implement/Config_shift_left_count_down/top_route_design.dcp are compatible
```

验证了配置之后，可以针对选择的板卡来生成对应的比特流。

将第一个配置读入内存：

```
open_checkpoint Implement/Config_shift_right_count_up_implement/top_route_design.dcp
```

然后为该设计生成完整和部分比特流，确保将比特流文件保存在与创建它们的完整设计检查点相关的唯一目录中。

```
write_bitstream -force -file Bitstreams/Config_RightUp.bit
```

可以看到，生成了三个比特流：

- `Config_RightUp.bit` - 这是启动后的完整设计比特流。右边的四个移位 led 将向右移动，左边的四个计数 led 将向上计数。
- `Config_RightUp_Pblock_inst_shift_partial.bit` - 这是 shift_right 模块的部分比特流。
- `Config_RightUp_Pblock_inst_count_partial.bit` - 这是count_up模块的部分比特流。

接着为第二个配置生成完整和部分比特流，将生成的比特文件保存在适当的文件夹中。

```
open_checkpoint Implement/Config_shift_left_count_down_import/top_route_design.dcp
write_bitstream -force -file Bitstreams/Config_LeftDown.bit
```

类似地，生成了三个比特流，只是拥有不同的名称。

此外，可以用灰盒生成一个完整的比特流，加上可重构模块的空白比特流。空白比特流可用于“擦除”现有配置以降低功耗。

```
open_checkpoint Checkpoint/static_route_design.dcp
update_design -cell inst_count -buffer_ports
update_design -cell inst_shift -buffer_ports
place_design
route_design
write_checkpoint -force Checkpoint/Config_greybox.dcp
write_bitstream -force -file Bitstreams/config_greybox.bit
close_project
```

基本配置比特流中对于这两个可重构分区都没有逻辑。这里的 update_design 命令为可重构分区的所有输出插入常量驱动程序（接地），因此这些输出不会浮动。灰盒表示插入了这些 LUT 后模块并非完全空的，与黑盒不同，黑盒将在该区域内外将网络对应引脚悬空。place_design 和 route_design 命令确保它们被完全实现。

**9. 部分重构 FPGA**

首先打开 Hardware Manager，选择设备后点击 Program device，在 `Bitstreams ` 文件夹中选择 `Config_RightUp.bit` 比特流，点击 OK 完成设备烧写。这与通常的设备烧写流程相似。

然后可以使用已创建的任何部分比特流部分地重新配置 FPGA 设备：

- Program device，在 `Bitstreams ` 文件夹中选择 `Config_LeftDown_pblock_inst_shift_partial.bit`，点击 OK 完成设备烧写。
- Program device，在 `Bitstreams ` 文件夹中选择 `Config_LeftDown_pblock_inst_count_partial.bit`，点击 OK 完成设备烧写。

可以观察到，原来正向计数的计数器现在在倒数，并且 led 的移动不受重新配置的影响。可以使用灰盒比特流来停止计数器或移位器的动作。

