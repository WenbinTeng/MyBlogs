---
title: fpga-dfx-script-flow
date: 2024-05-29 21:22:14
tags:
typora-root-url: fpga-dfx-script-flow
---
# FPGA DFX Script Flow

Vivado 提供的 Dynamic Function eXchange（DFX）功能是面向异构计算平台中模块级资源动态调度的重要机制，适用于多任务切换、系统热升级以及异构算力重调度等场景。本实验将介绍 Vivado 中基于 Tcl 脚本实现的 DFX 编译流程。以下是基本流程：

- 首先，独立综合所有静态模块（Static Modules）和每个可重构区域的变体（Variants for each Reconfigurable Partitions）；
- 然后，定义各个可重构区域的布局规划（Pblock），并实现该设计的初始配置；
- 接着，锁定设计的静态部分、使用变体更新可重构模块、重新运行实现可选配置；
- 最后，验证每个实现的可重构模块是否与设计的静态部分兼容，如果兼容，则生成比特流。

DFX 编译流程仍基于 Vivado 的典型设计流程：综合（Synthesis）- 实现（Implementation，包括布局与布线）- 比特流生成（Bitstream Generation）。在此基础上，DFX 设计要求将整个系统设计划分为静态部分与动态可重构设计，并分别对每种配置组合执行独立的实现流程。此外，还需引入额外的检查步骤，以确保静态设计在多个配置下保持一致性，从而保证运行时的正确性。

当前 Vivado 对 DFX 的支持以 Tcl 脚本为主，本文将介绍 Tcl 脚本流程的 DFX 完整实现路径。



### 1. 准备设计文件

从 Xilinx 官网下载官方 DFX 教程设计文件：

[下载链接](https://www.xilinx.com/member/forms/download/design-license.html?cid=03e48cb4-ba89-496d-a3d7-cbaa2302ef79&filename=ug947-vivado-partial-reconfiguration-tutorial.zip)

下载完成后，将文件解压至任意具备写权限的本地路径，然后进入解压后的 `led_shift_count_7s` 子目录。本实验中的所有步骤都将在该目录下完成。

### 2. 查看脚本功能

首先，我们查看设计目录中提供的脚本文件。根目录下包括两个主要 Tcl 脚本：

- `run_dfx.tcl`：执行 DFX 编译流程所需的最小脚本，定义了项目结构、模块划分、配置组合等关键内容；
- `advanced_settings.tcl`：工作流参数高级设置文件，通常不需更改。

##### 主要脚本说明

在 led_shift_count_7s 目录下的 run_dfx.tcl 是主要脚本，您可以在其中定义设计参数、设计源文件和设计结构。一般而言，如果需要编译自定义的 DFX 设计，这是唯一需要修改的脚本。有关 run_dfx.tcl、advanced_settings.tcl 以及底层脚本更详细的信息，请参阅位于 Tcl_HD 子目录中的 README.txt。

注意以下  run_dfx.tcl 脚本中的详细信息：

- 在“Define target demo board”下，您可以选择支持此设计的众多演示板卡之一。

  ```tcl
  ###############################################################
  ### Define target demo board (select one)
  ### Valid values: kc705 (default), vc707, vc709, ac701
  ###############################################################
  set xboard        "kc705"
  ```
  
- 在“flow control”下，您可以控制运行合成和实现的哪些阶段。在本教程中，只有综合是由脚本运行的，而实现、验证和比特流生成是交互运行的。如果需要通过脚本运行这些附加步骤，请将工作流变量（例如 run. prImpl）设置为1。

  ```tcl
  ###############################################################
  ###  Minimum settings required to run DFX flow:
  ###  1. Specify flow steps
  ###  2. Define target board
  ###  3. Identify source directories
  ###  4. Define static module
  ###  5. Define RPs, and their RM variants
  ###############################################################
  ####flow control (1 = run step , 0 = skip step)
  set run.topSynth       1 ;#synthesize static
  set run.rmSynth        1 ;#synthesize RM variants
  set run.dfxImpl        0 ;#implement each static + RM configuration
  set run.prVerify       0 ;#verify RMs are compatible with static
  set run.writeBitstream 0 ;#generate full and partial bitstreams
  ```
  
- “Output Directories”和“Input Directories”设置了设计源文件和结果文件所需的文件结构。您必须在这里反映对文件结构的任何更改。

  ```tcl
  ###############################################################
  ###  Run Settings
  ###############################################################
  ####Input Directories
  set srcDir     "./Sources"
  set rtlDir     "$srcDir/hdl"
  set prjDir     "$srcDir/prj"
  set xdcDir     "$srcDir/xdc"
  set coreDir    "$srcDir/cores"
  set netlistDir "$srcDir/netlist"
  
  ####Output Directories
  set synthDir  "./Synth"
  set implDir   "./Implement"
  set dcpDir    "./Checkpoint"
  set bitDir    "./Bitstreams"
  ```
  
- “Top Definition”（注：在脚本中是“Static Module Definition”部分）和“RP Module Definitions”（注：在脚本中是“RP & RM Definitions”部分）允许您引用设计中的所有源文件。“Top Definition”部分涵盖了静态设计所需的所有资源，包括约束和 IP。“RP Module Definitions”部分对可重构分区做了相同的操作，识别每个可重构分区并列出每个可重构分区的所有可重构模块变体。

  - 本教程的设计有两个可重构分区：inst_shift 和 inst_count。每个可重构分区有两个可重构模块变体。

  ```tcl
  ###############################################################
  ### Static Module Definition
  ###############################################################
  set top "top"
  
  ###############################################################
  ### RP & RM Definitions (Repeat for each RP)
  ### 1. Define Reconfigurable Partition (RP) name
  ### 2. Associate Reconfigurable Modules (RMs) to the RP
  ###############################################################
  set rp1 "shift"
  set rm_variants($rp1) "shift_right shift_left"
  set rp2 "count"
  set rm_variants($rp2) "count_up count_down"
  ```
  
- “Configuration Definition”（注：在脚本中是“RM Configurations”部分）定义了组成配置的静态和可重构模块集。

  - 本教程的设计在主脚本中定义了两个配置：config_shift_right_count_up_implementation 和 config_shift_left_count_down_import。
  - 您可以通过添加可重构模块或组合现有可重构模块来创建更多配置。

  ```tcl
  ########################################################################
  ### RM Configurations (Valid combinations of RM variants)
  ### 1. Define initial configuration: rm_config(initial)
  ### 2. Define additional configurations: rm_config(xyz)
  ########################################################################
  set module1_variant1 "shift_right"
  set module2_variant1 "count_up"
  set rm_config(initial)   "$rp1 $module1_variant1 $rp2 $module2_variant1"
  set module1_variant2 "shift_left"
  set module2_variant2 "count_down"
  set rm_config(reconfig1) "$rp1 $module1_variant2 $rp2 $module2_variant2"
  ```

##### 支撑脚本

在 Tcl_HD 子目录下，存在几个 Tcl 支持脚本。这些脚本由 run_dfx.tcl 调用，它们管理 DFX 工作流的特定细节。下面提供了其中几个关键脚本的详细信息：

- step.tcl：通过监视检查点来管理设计的当前状态。
- synthesize.tcl：管理有关综合阶段的所有细节。
- implement.tcl：管理有关模块实现阶段的所有细节。
- dfx_utils.tcl：管理有关 DFX 设计顶层实现的所有细节。
- run.tcl：为综合和实现启动实际运行。
- log_utils.tcl：处理流程中关键点上的报告文件创建



### 3. 综合设计

run_dfx.tcl 脚本自动化了本实验的综合阶段。脚本将迭代调用五次综合，一次用于静态顶层设计，四次用于四个可重构模块设计的综合。

1. 打开 Vivado Tcl Shell。

2. 在 Vivado Tcl Shell 中定位到 led_shift_count_7s。

3. 如果选择的是 KC705 以外的目标演示板，则需要在 run_dfx.tcl 脚本中修改 xboard 变量至目标演示板。

4. 在 Vivado Tcl Shell 中输入以下命令以运行 run_dfx.tcl 脚本：

   ```tcl
   source run_dfx.tcl -notrace
   ```

完成所有综合步骤之后，生成的 Synth 子目录下，每个以模块单独命名文件夹中可以找到对应模块的日志和报告文件，以及最终的检查点。这些日志文件包括：

- run.log：综合运行的汇总。
- command.log：打印脚本中所有运行步骤。
- critical.log：报告运行过程中产生的所有关键警告（critical warnings）。



### 4. 集成设计

当所有模块都完成综合并生成相应的检查点之后，就可以进行设计的集成了。在这里以及之后的步骤中，我们借助 IDE 内的交互界面来展示一些命令的作用，在实际的工作流程中完全可以使用 Tcl 脚本来完成所有的步骤。进行设计集成的具体步骤如下：

1. 打开 Vivado IDE。

2. 在 Vivado IDE 中的 Tcl 控制台中定位到 led_shift_count_7s。

3. 在 Tcl 控制台中设置目标芯片以及目标开发板的变量。

   ```tcl
   set part "xc7k325t-ffg900-2"
   set board "kc705"
   
   set part "xc7vx485t-ffg1761-2"
   set board "vc707"
   
   set part "xc7vx690t-ffg1761-2"
   set board "vc709"
   
   set part "xc7a200t-fbg676-2"
   set board "ac701"
   ```

4. 在 Tcl 控制台中创建一个内存工程。

   ```tcl
   create_project -in_memory -part $part
   ```

5. 加载静态设计对应的检查点。

   ```tcl
   add_files ./Synth/Static/top_synth.dcp
   ```

6. 加载顶层设计约束。

   ```tcl
   add_files ./Sources/xdc/top_io_$board.xdc
   set_property USED_IN {implementation} [get_files ./Sources/xdc/top_io_$board.xdc]
   ```

   这里的 xdc 约束文件只包含引脚位置和时钟约束，但不包含布局信息。

7. 分别加载 shift 和 count 模块综合生成的第一个检查点，即 shift_right 和 count_up 的检查点。

   ```tcl
   add_files ./Synth/shift_right/shift_synth.dcp
   set_property SCOPED_TO_CELLS {inst_shift} [get_files ./Synth/shift_right/shift_synth.dcp]
   add_files ./Synth/count_up/count_synth.dcp
   set_property SCOPED_TO_CELLS {inst_count} [get_files ./Synth/count_up/count_synth.dcp]
   ```

   这里的 SCOPED_TO_CELLS 属性确保对目标 cell 的正确分配。

8. 链接整个设计。

   ```tcl
   link_design -mode default -reconfig_partitions {inst_shift inst_count} -part $part -top top
   ```

9. 保存此集成设计的初始配置。

   ```tcl
   write_checkpoint -force ./Checkpoint/top_link_right_up.dcp
   ```



### 5. 建立设计布局规划

接下来，我们将创建一个布局规划来定义部分重构区域。具体步骤如下：

1. 在 Netlist 窗口中选择 inst_count 实例。右键单击，选择 Floorplanning > Draw Pblock ，或选择 Draw Pblock 工具栏按钮，在 X0Y3 时钟区域的左侧绘制一个高窄框。在这一点上，确切的大小和形状并不重要，但需要保持所绘制的框在时钟区域内。

   <img src="1.png" style="zoom:80%;" />

   虽然这个可重构模块只需要 CLB 资源，但是绘制的 Pblock 还包括 RAMB18、RAMB36 或 DSP48 资源，这是允许的。如果需要，可以使用 Pblock 属性窗口的 General 视图来添加这些属性。Statistics 视图显示当前加载的可重构模块的资源需求。

2. 在 Properties 视图中，选择 RESET_AFTER_RECONFIG 的复选框，以便在重新配置完成后使用该模块中逻辑的专用初始化。

3. 然后对 inst_shift 实例重复步骤 1 和 2，绘制一个新的 Pblock 在时钟域 X1Y1 的右侧。这个可重构模块包括 Block RAM 实例，因此必须包含资源类型。如果忽略这一需求，则 Statistics 视图中的 RAMB 详细信息将以红色显示。

   <img src="2.png" style="zoom:80%;" />

4. 选择 Reports > Report DRC 运行 DFX 设计规则检查。您可以取消选中 All Rules，然后仅选中 Dynamic Function eXchange，将此报告集中在 DFX DRCs 上。

   <img src="3.png" style="zoom:80%;" />

   运行检查后会报告一个或两个 DRC 违规，分别有两种解决它们的办法。在本实验中，将分别使用这两种方法解决 inst_shift 和 inst_count 实例中 DRC 违规的问题。

5. 第一个 DRC 违规是一个错误，HDPR-10，报告 RESET_AFTER_RECONFIG 要求 Pblock 帧对齐。要解决第一个 DRC 错误，请确保 Pblock 的高度与时钟区域边界对齐。将 inst_shift 的 Pblock 拉伸顶部和底部边缘以匹配 X1Y1 的时钟区域边界，如下图所示。

   <img src="4.png" style="zoom:80%;" />

6. 第二个可能的 DRC 违规是一个警告，HDPR-26，报告 Pblock 的左边缘或右边缘在不适当的边界上终止。左边缘或右边缘不能分割 INT 列。如果要手动避免此 DRC 警告，需要将 inst_shift 的 Pblock 放大，查看发生违规的边缘。将 Pblock 的边框向左或向右移动一列，如下图的黄色箭头所示，这样它就会落在两个资源类型（例如 CLB-CLB 或 CLB-RAMB）之间，而不是落在 CLB-INT 或 RAMB-INT 之间，如下图所示。

   <img src="5.png" style="zoom:80%;" />

7. 再次运行 DRC，以确认已经解决了对应的错误和警告。

8. 代替手动调整可重构 Pblock 的大小和形状的另一种方法是使用 SNAPPING_MODE 特性。此功能自动调整边缘以与合法边界对齐。如果选择 RESET_AFTER_RECONFIG 特性，它将使 Pblock 更高，与时钟区域边界对齐；同时，也会使 Pblock 变窄，根据需要调整左边缘和右边缘。注意，如果使用 SNAPPING_MODE 对 Pblock 进行更改，可用资源的数量和类型就会改变。在 Device 窗口中选择 inst_count 的 Pblock，并在 Pblock Properties 窗口的 Properties 视图中，将 SNAPPING_MODE 的值从 OFF 更改为 ROUTING （或 ON）。

9. 再次运行 DRC，确认所有错误都已解决。可能仍然会报告建议信息，特别是如果 Pblock 位于设备的边缘附近时。

10. 保存这些 Pblock 和相关属性：

    ```
    write_xdc ./Sources/xdc/top_all.xdc
    ```

    这将导出设计中的所有当前约束，包括之前从 top_io_$board.xdc 导入的约束。这些约束可以在它们自己的 XDC 文件中进行管理，也可以在运行脚本中进行管理（通常与 HD.RECONFIGURABLE 一起使用）。

    另外，Pblock 约束本身也可以被提取并单独管理。有一个 Tcl 脚本流程可以帮助执行此任务：

    1. 读取并执行 hd_utils.tcl 脚本中的命令。

       ```
       source ./Tcl_HD/hd_utils.tcl
       ```

    2. 使用 export_pblocks 命令来输出 Pblock 的约束信息。

       ```tcl
       export_pblocks -file ./Sources/xdc/pblocks.xdc
       ```

       此操作为设计中的两个 Pblock 写入 Pblock 约束信息。如果需要，请使用 -pblocks 选项选择其中一个。



### 6. 实现第一个配置

接下来，我们将对准备的静态设计进行实现（布局布线），以供可重构模块进行重用。以下是具体步骤：

1. 优化、布局和布线静态设计。

   ```tcl
   opt_design
   place_design
   route_design
   ```

   运行结束之后，在 Device 视图中检查设计状态，如下图所示。place_design 之后需要注意的一点是，其引入了 Partition Pins，这些是静态逻辑和可重构逻辑之间的物理接口点，即可重构模块的每个 I/O 必须经过的互连块中的锚点。它们在布局的设计视图中显示为白框。

   对于 pblock_shift，它们出现在该 Pblock 的顶部，因为到静态区域的连接就在设备上该区域的 Pblock 外部。对于 pblock_count，它们出现在用户定义区域之外，因为 SNAPPING_MODE 垂直收集更多帧以添加到可重构分区。

   <img src="6.png" style="zoom:80%;" />

2. 如果您想在 GUI 中找到这些 Partition Pins：

   1.  在 Netlist 窗口中选择该可重构模块。
   2.  在 Cell Properties 窗口中选择 Cell Pins 选项。

3. 选择任一引脚就可以将其高亮显示，如果您想高亮显示所有引脚，请使用 Ctrl+A 或者：

   ```tcl
   select_objects [get_pins inst_shift/*]
   ```

4. 使用布线资源工具栏按钮在抽象布线信息和实际布线信息之间切换，并更改布线资源本身的可见性。在这一视图上，设计中的所有网络都是完全布线的。

   <img src="7.png" style="zoom:80%;" />

以下步骤将保存第一个配置的实现结果：

1. 保存完整的设计检查点并创建报告文件。

   ```tcl
   write_checkpoint -force Implement/Config_shift_right_count_up_implement/top_route_design.dcp 
   
   report_utilization -file Implement/Config_shift_right_count_up_implement/top_utilization.rpt
   
   report_timing_summary -file Implement/Config_shift_right_count_up_implement/top_timing_summary.rpt
   ```

2. （可选）为每个可重构模块保存检查点。

   ```tcl
   write_checkpoint -force -cell inst_shift Checkpoint/shift_right_route_design.dcp
   write_checkpoint -force -cell inst_count Checkpoint/count_up_route_design.dcp
   ```

   在此阶段，您已经创建了一个完全实现的 DFX 设计，您可以基于此设计生成完整比特流与部分比特流。

3. 此配置中的静态部分可以被后续的配置重用。为了隔离静态设计，需要移除当前设计中的可重构模块。此外还需要确保启用布线资源，并放大到具有可重构分区引脚的互联分块。通过以下命令清除可重构模块的逻辑：

   ```tcl
   update_design -cell inst_shift -black_box
   update_design -cell inst_count -black_box
   ```

   执行这些命令会导致设计发生以下更改：

   - 完全布线网络（绿色）的数量减少。
   - inst_shift 和 inst_count 现在在 Netlist 视图中显示为空。

   下图显示了 inst_shift 模块在执行 update_design 命令之前的状态。

   <img src="8.png" style="zoom:80%;" />

   下图显示了 inst_shift 模块在执行 update_design 命令之后的状态。

   <img src="9.png" style="zoom:80%;" />

4. 执行以下命令以锁定所有布局和布线：

   ```tcl
   lock_design -level routing
   ```

   由于在 lock_design 命令中没有指定任何 cell，所以内存中的整个设计（目前由静态设计和黑盒组成）都会受到影响。所有已经布线的网络现在显示为锁定，如下图中虚线所示。所有布局的组件从蓝色变为橙色，表示它们也被锁定。

   <img src="10.png" style="zoom:80%;" />

5. 执行以下命令以输出当前（由静态设计和黑盒组成）的静态配置的检查点：

   ```tcl
   write_checkpoint -force Checkpoint/static_route_design.dcp
   ```

6. 在进行下一个配置之前，关闭此设计：

   ```tcl
   close_project
   ```



### 7. 实现第二个配置

现在静态设计结果已经建立并锁定，我们将用它作为实现其他可重构模块的上下文。以下是具体步骤：

1. 建立一个新的内存工程。

   ```tcl
   create_project -in_memory -part $part
   ```

2. 加载静态设计。

   ```tcl
   add_files ./Checkpoint/static_route_design.dcp
   ```

3. 分别加载 shift 和 count 模块综合生成的第二个检查点，即 shift_left 和 count_down 模块的检查点。

   ```tcl
   add_files ./Synth/shift_left/shift_synth.dcp
   set_property SCOPED_TO_CELLS {inst_shift} [get_files ./Synth/shift_left/shift_synth.dcp]
   add_files ./Synth/count_down/count_synth.dcp
   set_property SCOPED_TO_CELLS {inst_count} [get_files ./Synth/count_down/count_synth.dcp]
   ```

4. 链接整个设计。

   ```tcl
   link_design -mode default -reconfig_partitions {inst_shift inst_count} -part $part -top top
   ```

   此时，已加载完整配置。然而，此时的静态设计是已布线的和锁定的，可重构模块的检查点仍然是一个逻辑网表。所以，之后的布局布线过程仅适用于可重构模块。

5. 优化、布局和布线新的可重构模块。

   ```tcl
   opt_design 
   place_design 
   route_design
   ```

   此时整个设计再次被完全实现，现在有了新的可重构模块变体。布线是由虚线（锁定的）和实线（新布线的）组成的混合体，如下图所示。

   <img src="11.png" style="zoom:80%;" />

以下步骤将保存第二个配置的实现结果：

1. 保存完整的设计检查点并创建报告文件。

   ```tcl
   write_checkpoint -force Implement/Config_shift_left_count_down_import/top_route_design.dcp
   
   report_utilization -file Implement/Config_shift_left_count_down_import/top_utilization.rpt
   
   report_timing_summary -file Implement/Config_shift_left_count_down_import/top_timing_summary.rpt
   ```

2. （可选）为每个可重构模块保存检查点。

   ```tcl
   write_checkpoint -force -cell inst_shift Checkpoint/shift_left_route_design.dcp
   write_checkpoint -force -cell inst_count Checkpoint/count_down_route_design.dcp
   ```

   在此阶段，您已经实现了静态设计和所有可重构模块的变体。如果设计的可重构分区还包含更多的可重构模块设计，则重复这个过程。



### 8. 检查结果

在 IDE 中打开已布线的配置后，运行一些可视化脚本可以突出显示分块和网络。这些脚本标识了为 DFX 分配的资源，并且是自动生成的。

1. 在 Tcl 控制台中，在工作目录中执行以下命令：

   ```tcl
   source hd_visual/pblock_inst_shift_AllTiles.tcl
   highlight_objects -color blue [get_selected_objects]
   ```

2. 在 Device 视图中点击某处以取消选中帧（或者输入 unselect_objects），然后执行以下命令：

   ```tcl
   source hd_visual/pblock_inst_count_AllTiles.tcl
   highlight_objects -color yellow [get_selected_objects]
   ```

   在分区帧在 Device 视图中突出显示，如图所示：

   <img src="12.png" style="zoom:80%;" />

   这些高亮的块代表会被发送到比特流生成的配置帧，以创建部分比特流。如上图所示，SNAPPING_MODE 功能调整了 pblock_count 的所有四个边缘，以考虑 RESET_AFTER_RECONFIG 与合法的可重构分区宽度。

   其他分块的脚本都是这些脚本的变体。如果您没有创建与时钟区域边界垂直对齐的 Pblock，则 FrameTiles 脚本将突出显示 Pblock，而 AllTiles 脚本将扩展这些分块到完整的可重构帧高度。请注意，这些脚本会在未选择帧类型的地方（例如：全局时钟）留下间隙。

   GlitchTiles 脚本是帧站点的一个子集，避免显示专用硅资源。

3. 关闭当前设计：

   ```tcl
   close_project
   ```



### 9. 生成比特流

在生成比特流之前，我们需要验证所有配置，以确保每个配置的静态部分完全相同，从而使生成的比特流在硅中安全使用。PR Verify 功能检查完整的静态设计，包括分区引脚，确认它们是否完全相同。可重构模块内的布局与布线不会进行检查。以下是验证配置的具体步骤：

1. 在 Tcl 控制台中运行 pr_verify 命令：

   ```tcl
   pr_verify Implement/Config_shift_right_count_up_implement/top_route_design.dcp Implement/Config_shift_left_count_down_import/top_route_design.dcp
   ```

   如果成功，此命令将返回以下信息。

   ```
   INFO: [Vivado 12-3253] PR_VERIFY: check points Implement/Config_shift_right_count_up/
   top_route_design.dcp and Implement/Config_shift_left_count_down/top_route_design.dcp are compatible
   ```

   默认情况下，仅报告第一个不匹配项（如果有）。要查看所有不匹配项，请使用 -full_check 选项。

2. 关闭项目：

   ```tcl
   close_project
   ```

如果配置验证成功，则可以生成目标演示板对应的比特流。以下是生成比特流的具体步骤：

1. 将第一个配置读入内存：

   ```tcl
   open_checkpoint Implement/Config_shift_right_count_up_implement/top_route_design.dcp
   ```

2. 生成此设计的完整比特流和部分比特流。确保将比特流文件保存在与创建的完整设计检查点相同的唯一目录中：

   ```tcl
   write_bitstream -force -file Bitstreams/Config_RightUp.bit
   close_project
   ```

   此时，将创建三个比特流：

   - Config_RightUp.bit：这是系统上电时使用的完整比特流。右侧的四个移位 LED 将向右移动，左侧的四个计数 LED 将向上计数。
   - Config_RightUp_Pblock_inst_shift_partial.bit：这是 shift_right 模块的部分比特流文件。
   - Config_RightUp_Pblock_inst_count_partial.bit：这是 count_up 模块的部分比特流文件。

   如果需要指定部分比特流的名称，可以使用 -file 选项以提供基本名称，并附加可重构单元的 Pblock 名称。在 advanced_settings.tcl 脚本中可以进行相关的配置。

3. 为第二个配置生成完整比特流和部分比特流，并将生成的位文件保存在相应的文件夹中：

   ```tcl
   open_checkpoint Implement/Config_shift_left_count_down_import/top_route_design.dcp
   write_bitstream -force -file Bitstreams/Config_LeftDown.bit
   close_project
   ```

   类似地，生成了三个比特流，只是拥有不同的名称。

4. 生成带有灰盒的完整比特流文件，以及可重构区域内对应的空白比特流文件。这些空白比特流可用于“擦除”现有配置以降低功耗。

   ```tcl
   open_checkpoint Checkpoint/static_route_design.dcp
   update_design -cell inst_count -buffer_ports
   update_design -cell inst_shift -buffer_ports
   place_design
   route_design
   write_checkpoint -force Checkpoint/Config_greybox.dcp
   write_bitstream -force -file Bitstreams/config_greybox.bit
   close_project
   ```

   这个基础的配置没有为可重构分区提供逻辑。这里的 update_design 命令为可重构分区的所有输出插入常量驱动器（接地），因此这些输出不会变化。“灰盒”表示插入了这些 LUT 后模块并非完全空的，与“黑盒”不同，“黑盒”将在该区域内外将网络对应引脚悬空。place_design 和 route_design 命令确保它们被完全实现。



### 10. 部分重构 FPGA

首先打开 Hardware Manager，选择设备后点击 Program device，在 Bitstreams  文件夹中选择 Config_RightUp.bit 比特流，点击 OK 完成设备烧写。这与通常的设备烧写流程相似。然后可以使用已创建的任何部分比特流部分地重新配置 FPGA 设备：

- Program device，在 Bitstreams  文件夹中选择 Config_LeftDown_pblock_inst_shift_partial.bit，点击 OK 完成设备烧写。
- Program device，在 Bitstreams  文件夹中选择 Config_LeftDown_pblock_inst_count_partial.bit，点击 OK 完成设备烧写。

可以观察到，原来正向计数的计数器现在在倒数，并且 led 的移动不受重新配置的影响。可以使用灰盒比特流来停止计数器或移位器的动作。

