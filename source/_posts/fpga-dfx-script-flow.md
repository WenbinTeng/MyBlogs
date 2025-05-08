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
- `advanced_settings.tcl`：设置 DFX 工作流程参数的高级设置文件，通常不需更改。

##### 主要脚本说明

`led_shift_count_7s` 目录中的 `run_dfx.tcl` 是本设计的核心脚本。该脚本允许用户定义设计参数、设计源文件和设计结构，以及 DFX 的执行流程控制。在多数情况下，用户仅需修改该脚本即可适配新的 DFX 设计。同一目录下的 `advanced_settings.tcl` 脚本是 DFX 流程的高级设置文件，其中可以定义底层辅助脚本调用参数、开发板参数以及运行参数等。

如需了解 `run_dfx.tcl`、`advanced_settings.tcl` 以及辅助脚本的具体功能，可以参考 `Tcl_HD/` 子目录下的 `README.txt` 文件。

以下是 `run_dfx.tcl` 脚本中值得关注的关键配置：

- **定义目标开发板**：在 "Define target demo board" 段落中设置 `xboard` 变量，以指定用于实验的开发板。

    ```tcl
    ###############################################################
    ### Define target demo board (select one)
    ### Valid values: kc705 (default), vc707, vc709, ac701
    ###############################################################
    set xboard        "kc705"
    ```

- **DFX 流程控制**：通过设置 `run.xxx` 变量控制各阶段的执行（1 表示执行，0 表示跳过）。默认脚本仅执行综合阶段，后续实现、验证和比特流生成将在交互模式下完成。如需启用这些步骤，可将对应变量设为 1：

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

- **输入/输出路径配置**：配置设计文件的位置及各阶段输出的保存路径。如目录结构发生更改，需同步修改对应变量。

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

- **模块定义**：在“Static Module Definition”段落中设置静态顶层模块，这个模块中涵盖了静态设计所需的所有资源，包括约束和 IP。在“RP Module Definitions”段落设置每个可重构区域（Reconfigurable Partitions, RP）和对应的所有可重构模块（Reconfigurable Modules, RM）。脚本中定义了两个可重构区域 `shift` 和 `count`，以及区域中对应的可重构模块 `shift_right` 与 `shift_left`、`count_up` 与 `count_down`。

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

- **配置组合定义**：在 "RM Configurations" 段落中定义配置名称与所使用的可重构模块组合。脚本中提供了两个配置示例，可按需添加更多组合。脚本中定义了两个配置：`config_shift_right_count_up_implementation ` 与 `config_shift_left_count_down_import`。

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

##### 辅助脚本说明

`Tcl_HD/` 目录下包含若干被主要脚本调用的支撑 Tcl 脚本文件，它们管理着 DFX 工作流的特定细节。下面是其中几个关键脚本的详细信息：

- `step.tcl`：管理各阶段状态并监控检查点；
- `synthesize.tcl`：控制模块综合阶段；
- `implement.tcl`：处理配置实现阶段；
- `dfx_utils.tcl`：管理有关 DFX 设计顶层实现的所有细节；
- `run.tcl`：统一调用综合与实现命令；
- `log_utils.tcl`：处理各阶段报告与日志输出。



### 3. 综合设计

`run_dfx.tcl` 脚本自动化了本实验的综合阶段。该脚本将迭代执行五次综合任务：一次用于静态顶层设计，其余四次分别用于每个 RM 的综合。

1. 启动 Vivado Tcl Shell。

2. 将当前工作目录切换至 `led_shift_count_7s`。

3. 若使用的开发板非默认的 KC705，请在 `run_dfx.tcl` 中将 `xboard` 变量修改为目标开发板型号。

4. 在 Tcl 终端中执行以下命令以启动主要脚本：

   ```tcl
   source run_dfx.tcl -notrace
   ```

完成所有综合阶段之后，每个模块的结果将存放于 `Synth/` 子目录下，以模块名命名的子文件夹中可找到相应的日志与报告文件，以及最终生成的设计检查点（Design Check Point，DCP）。主要日志包括：

- `run.log`：记录综合阶段的运行汇总信息；
- `command.log`：打印脚本执行的完整 Tcl 命令序列；
- `critical.log`：汇总综合过程中出现的所有关键警告（Critical Warnings）。



### 4. 集成设计

在完成所有模块的综合并生成对应的 DCP 文件后，即可进行初始设计的集成。以下步骤可通过 Vivado GUI 或 Tcl 脚本完成，本节将结合 IDE 展示一些命令的作用，实际工程中也可完全采用脚本自动化。集成流程如下：

1. 启动 Vivado IDE。

2. 在 Tcl 控制台中切换至 `led_shift_count_7s` 目录。

3. 设置目标器件型号和开发板变量。

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

4. 创建一个内存工程（in-memory project）。

   ```tcl
   create_project -in_memory -part $part
   ```

5. 加载静态顶层设计综合生成的 DCP 文件。

   ```tcl
   add_files ./Synth/Static/top_synth.dcp
   ```

6. 加载板卡对应的顶层约束文件（Xilinx Design Constraints，XDC）。该约束文件包含引脚绑定和时钟定义，但不包含布局信息。

   ```tcl
   add_files ./Sources/xdc/top_io_$board.xdc
   set_property USED_IN {implementation} [get_files ./Sources/xdc/top_io_$board.xdc]
   ```

7. 分别加载 shift 和 count 模块综合生成的第一个 DCP，即 shift_right 和 count_up 的 DCP 文件。其中，`SCOPED_TO_CELLS` 属性确保对目标 Cell 的正确分配。

   ```tcl
   add_files ./Synth/shift_right/shift_synth.dcp
   set_property SCOPED_TO_CELLS {inst_shift} [get_files ./Synth/shift_right/shift_synth.dcp]
   add_files ./Synth/count_up/count_synth.dcp
   set_property SCOPED_TO_CELLS {inst_count} [get_files ./Synth/count_up/count_synth.dcp]
   ```

8. 调用 `link_design` 完成静态顶层设计与动态可重构模块的结构链接，并指定本次配置包含的所有 RP。

   ```tcl
   link_design -mode default -reconfig_partitions {inst_shift inst_count} -part $part -top top
   ```

9. 使用 `write_checkpoint` 保存当前集成状态作为后续实现阶段的输入。

   ```tcl
   write_checkpoint -force ./Checkpoint/top_link_right_up.dcp
   ```



### 5. 建立设计布局规划

本节将为每个 RP 创建对应的布局规划（Pblock），用于定义它们在 FPGA 器件上的物理位置和资源边界。具体步骤如下：

1. 在 Netlist 视图中，选择 `inst_count` 实例。右键点击，选择 *Floorplanning → Draw Pblock*，或者使用工具栏上的 Draw Pblock 按钮，在 `X0Y3` 所在的时钟区域左侧手动绘制一个高而窄的矩形区域。这里，Pblock 的具体大小与形状尚不关键，但必须完全落在一个时钟区域内部。

   <img src="1.png" style="zoom:80%;" />

   虽然这个可重构模块只需要 CLB 资源，但是绘制的 Pblock 还包括 RAMB18、RAMB36 或 DSP48 资源，这是允许的。如果需要，可以使用 Pblock 属性窗口的 General 视图来添加这些属性。Statistics 视图显示当前加载的可重构模块的资源需求。

2. 在 Properties 视图中，选择 `RESET_AFTER_RECONFIG` 复选框，以确保在完成动态重构后自动初始化该模块逻辑的初始状态。

2. 对 `inst_shift` 实例重复上述步骤 1 和 2，在时钟区域 `X1Y1` 的右侧绘制第二个 Pblock。由于该模块包含 Block RAM 实例，因此其对应的 Pblock 必须覆盖足够的 BRAM 资源，否则 Statistics 面板中的 RAMB 项将以红色高亮显示，提示资源不足。

   <img src="2.png" style="zoom:80%;" />
   
2. 打开 Reports 菜单，选择 *Report DRC*，运行 DFX 设计规则检查。可取消勾选 “All Rules”，仅选择 “Dynamic Function eXchange” 类别，以聚焦与 DFX 相关的 DRC 报告。

   <img src="3.png" style="zoom:80%;" />
   
   运行后通常会触发一到两个 DRC 违规，本实验将分别对两个实例演示如何处理这些典型违规。
   
2. 第一个 DRC 违规是错误，HDPR-10，指出启用了 `RESET_AFTER_RECONFIG` 的 Pblock 必须沿时钟区域边界对齐。为修复该问题，需要手动调整 `inst_shift` 的 Pblock，使其上边界与下边界严格贴合 `X1Y1` 时钟区域的顶部与底部，如下图所示。

   <img src="4.png" style="zoom:80%;" />
   
2. 第二个可能出现的 DRC 违规是一个警告，HDPR-26，指出 Pblock 的左右边缘没有落在合法的列边界上。Vivado DFX 要求 Pblock 的左右边界不得横跨 INT 列（内部互连列）。解决方案是放大视图定位违规边界，然后将 Pblock 左/右边缘微调至两个合法资源（如 CLB-CLB 或 CLB-RAMB）之间的边界，而不是落在 CLB-INT 或 RAMB-INT 之间，如下图所示。

   <img src="5.png" style="zoom:80%;" />
   
2. 修正后再次运行 DRC 检查，确认所有错误与警告已被清除。

2. 除手动微调边界外，也可使用 `SNAPPING_MODE` 属性自动修正 Pblock 的对齐方式。在 Device 窗口中选择对应的 Pblock，并在 Pblock Properties 窗口的 Properties 视图中，将 `SNAPPING_MODE` 的值从 `OFF` 更改为 `ROUTING` （或 `ON`）。设置该属性将自动根据时钟区域与资源列边界调整 Pblock 尺寸，确保其满足重构对齐要求。特别地，若启用 `RESET_AFTER_RECONFIG`，Vivado 会强制拉高 Pblock，自动扩展其高度覆盖完整时钟区域。需要注意，使用 `SNAPPING_MODE` 后，Pblock 的可用资源数量和类型可能会发生变化，因此建议在设置后重新检查其资源布局。

2. 最终再次执行 DRC，确认所有限制已满足。如果 Pblock 接近器件边缘，可能仍会看到部分建议类信息（Info），但不会影响设计有效性。

2. 使用以下命令保存当前的布局规划与所有约束设置。

   ```tcl
   write_xdc ./Sources/xdc/top_all.xdc
   ```
   
   此命令将导出当前设计中所有生效的 XDC 约束文件，包括早前加载的 `top_io_$board.xdc` 中的引脚与时钟约束。这些约束可以在它们自己的 XDC 约束文件中进行管理，也可以在运行脚本中进行管理（通常与 `HD.RECONFIGURABLE` 一起使用）。
   
   如果需要单独提取单独的 Pblock 约束，可以使用 `hd_utils.tcl` 脚本帮助完成该操作：
   
   1. 读取并执行 `hd_utils.tcl` 脚本中的命令。
   
      ```tcl
      source ./Tcl_HD/hd_utils.tcl
      ```
   
   2. 使用 `export_pblocks` 命令来输出 Pblock 的约束信息。
   
      ```tcl
      export_pblocks -file ./Sources/xdc/pblocks.xdc
      ```
   
      该命令将为设计中的每个 Pblock 生成约束语句。你也可以使用 `-pblocks` 选项指定导出其中某一个。



### 6. 实现第一个配置

本节将对当前的静态设计及其所包含的 RM 进行布局与布线，从而完成第一个完整配置的实现结果。该配置将在后续其他 RM 变体的实现流程中被重用。实现的具体步骤如下：

1. 优化、布局和布线静态设计。

   ```tcl
   opt_design
   place_design
   route_design
   ```

   运行结束后可以在 Device 视图中检查设计状态，如下图所示。Vivado 会在静态设计与 RM 之间自动生成分区引脚（Partition Pins），这些引脚作为物理接口标识 RP 的边界，是 RM 的每个 I/O 必须经过的互联块中的锚点。它们通常在布局视图中以白色框标出：

   - 对于 `pblock_shift` 区域，Partition Pins 通常出现在 Pblock 的顶部，因为到静态区域的连接就在设备上该区域的 Pblock 外部。
   - 对于 `pblock_count` 区域，如果启用了 `SNAPPING_MODE`，Pblock 会自动在垂直方向上进行扩展，因此 Partition Pins 可能会出现在用户定义的区域之外。

   <img src="6.png" style="zoom:80%;" />

2. 如果希望通过 GUI 高亮显示某个可重构模块的 Partition Pins，可在 Netlist 视图中选中目标实例，在其 Cell Properties 窗口中切换到 *Cell Pins* 选项卡。

3. 若希望高亮全部引脚，可通过 Ctrl+A 或 Tcl 命令实现：

   ```tcl
   select_objects [get_pins inst_shift/*]
   ```

4. 在 Vivado 中使用布线资源工具栏按钮，可以在抽象布线信息和实际布线信息显示模式之间切换。在实际布线信息显示模式中，此时设计中的所有网络（Nets）应该是完全布线的。

##### 保存配置实现结果

布局布线完成后，需要保存设计检查点（DCP），并生成相关报告。

1. 输出完整配置的实现 DCP，并生成资源利用率与时序报告。

   ```tcl
   write_checkpoint -force Implement/Config_shift_right_count_up_implement/top_route_design.dcp 
   
   report_utilization -file Implement/Config_shift_right_count_up_implement/top_utilization.rpt
   
   report_timing_summary -file Implement/Config_shift_right_count_up_implement/top_timing_summary.rpt
   ```

2. （可选）为每个 RM 单独生成检查点，以便后续分析或复用。

   ```tcl
   write_checkpoint -force -cell inst_shift Checkpoint/shift_right_route_design.dcp
   write_checkpoint -force -cell inst_count Checkpoint/count_up_route_design.dcp
   ```

   至此，我们已完成第一个配置的完整实现，可以基于此实现生成完整比特流与部分比特流。

**锁定静态设计**

由于其他配置需要在同一个静态设计上实现，因此需要对该配置中的静态设计部分进行隔离并锁定。

1. 首先需要将当前配置中的 RM 移除，并配置为黑盒（Black Box）以移除内部逻辑。此外，还需要确保启用布线资源，并缩放到具有可重构分区引脚的互联分块。执行以下命令以移除 RM。

   ```tcl
   update_design -cell inst_shift -black_box
   update_design -cell inst_count -black_box
   ```

   执行上述命令之后：

   - 布线视图中原属于 RM 的网络布线（绿色）将被清除；
   - `inst_shift` 和 `inst_count` 现在在 Netlist 视图中显示为空。

   下图显示了 `inst_shift` 实例在执行 `update_design` 命令之前的状态。

   <img src="7.png" style="zoom:80%;" />

   下图显示了 `inst_shift` 实例在执行 `update_design` 命令之后的状态。

   <img src="8.png" style="zoom:80%;" />

2. 使用 `lock_design` 命令以锁定当前的布线状态。

   ```tcl
   lock_design -level routing
   ```

   由于未指定具体 cell，该命令将锁定设计中所有已布线对象，包括静态设计与 Partition Pins。被锁定的组件在布局视图中将由蓝色变为橙色，布线由实线变为虚线。

   <img src="9.png" style="zoom:80%;" />

3. 将当前设计保存为检查点以供后续使用。

   ```tcl
   write_checkpoint -force Checkpoint/static_route_design.dcp
   ```

4. 关闭当前项目。

   ```tcl
   close_project
   ```

   

### 7. 实现第二个配置

在第一个配置中，我们已完成了实现步骤，还提取了配置中的静态设计并锁定。现在，我们将基于已锁定的静态设计，加载新的 RM 变体，构建第二个配置，并完成该配置的实现。实现的具体步骤如下：

1. 创建一个内存工程（in-memory project）。

   ```tcl
   create_project -in_memory -part $part
   ```

2. 加载上一节中导出的静态设计 DCP。

   ```tcl
   add_files ./Checkpoint/static_route_design.dcp
   ```

3. 分别加载 `shift` 和 `count` 模块综合生成的第二个检查点，即 `shift_left` 和 `count_down` 模块的检查点。

   ```tcl
   add_files ./Synth/shift_left/shift_synth.dcp
   set_property SCOPED_TO_CELLS {inst_shift} [get_files ./Synth/shift_left/shift_synth.dcp]
   add_files ./Synth/count_down/count_synth.dcp
   set_property SCOPED_TO_CELLS {inst_count} [get_files ./Synth/count_down/count_synth.dcp]
   ```

4. 使用 `link_design` 命令连接整个新的配置。

   ```tcl
   link_design -mode default -reconfig_partitions {inst_shift inst_count} -part $part -top top
   ```

   此时，设计中的静态设计部分是已布线的和已锁定的，而 RM 的检查点仍然是逻辑网表。因此后续的布局布线操作仅会作用于新引入的两个 RM。

5. 优化、布局和布线新的 RM。

   ```tcl
   opt_design 
   place_design 
   route_design
   ```

   运行结束后，静态设计部分仍然是虚线以表示锁定状态，新增的 RM 布线则以实线显示，如下图所示。

   <img src="10.png" style="zoom:80%;" />

##### 保存配置实现结果

与第一个配置类似，我们也需要为该配置生成检查点和报告。

1. 保存完整配置的实现状态及相关报告：

   ```tcl
   write_checkpoint -force Implement/Config_shift_left_count_down_import/top_route_design.dcp
   
   report_utilization -file Implement/Config_shift_left_count_down_import/top_utilization.rpt
   
   report_timing_summary -file Implement/Config_shift_left_count_down_import/top_timing_summary.rpt
   ```

2. （可选）为每个 RM 单独生成检查点，以便后续分析或复用。

   ```tcl
   write_checkpoint -force -cell inst_shift Checkpoint/shift_left_route_design.dcp
   write_checkpoint -force -cell inst_count Checkpoint/count_down_route_design.dcp
   ```

   至此，我们已经完整实现了第二个配置，包含了静态设计和新的 RM 变体。如果设计的 RP 还包含更多的 RM 变体，则重复这个过程。



### 8. 检查结果

完成多个配置的布局布线之后，为确保静态区域与 RP 之间的边界定义合理、资源分配准确，我们可以借助一些可视化脚本对各 Pblock 的实际帧资源使用情况进行验证。

1. 在 Tcl 控制台中，运行以下命令以加载并高亮 `inst_shift` 的物理帧：

   ```tcl
   source hd_visual/pblock_inst_shift_AllTiles.tcl
   highlight_objects -color blue [get_selected_objects]
   ```

2. 取消选中当前帧（或者输入 `unselect_objects`），然后执行以下命令以加载并高亮 `inst_count` 的物理帧：

   ```tcl
   source hd_visual/pblock_inst_count_AllTiles.tcl
   highlight_objects -color yellow [get_selected_objects]
   ```
   

执行上述命令后，Vivado 会在 Device 视图中以对应颜色高亮显示每个 Pblock 的所有逻辑帧区域。这些被高亮的区域即为后续比特流生成过程中参与部分重构的目标区域。特别需要注意的是：若某个 Pblock 启用了 `RESET_AFTER_RECONFIG` 或 `SNAPPING_MODE` 属性，其边界将自动扩展以匹配完整时钟区域或帧对齐需求。因此，Pblock 的实际物理范围可能超出用户原先绘制的矩形。

其他脚本的说明：

- `AllTiles.tcl`：显示所有类型的可编程资源；
- `FrameTiles.tcl`：仅显示帧对齐的结果；
- `GlitchTiles.tcl`：排除掉部分专用硅资源（如全局时钟）的显示。

若 Pblock 未沿合法边界划分（如未完全对齐时钟区域或跨越 INT 列），这些脚本将帮助识别重构区域中资源使用的异常情况，便于后续进行重新规划或调优。

3. 关闭当前工程：

   ```tcl
   close_project
   ```



### 9. 生成比特流

在生成比特流之前，我们需要验证所有的配置，确保每个配置中的所有静态部分完全相同，从而使生成的比特流在硅中安全使用。Vivado 的 PR Verify 功能可以帮助检查完整的静态设计，包括布局布线、分区引脚等关键的属性完全一致。以下是验证配置的具体步骤：

1. 在 Tcl 控制台中运行 `pr_verify` 命令：

   ```tcl
   pr_verify Implement/Config_shift_right_count_up_implement/top_route_design.dcp Implement/Config_shift_left_count_down_import/top_route_design.dcp
   ```

   如果成功，此命令将返回以下信息。

   ```
   INFO: [Vivado 12-3253] PR_VERIFY: check points Implement/Config_shift_right_count_up/
   top_route_design.dcp and Implement/Config_shift_left_count_down/top_route_design.dcp are compatible
   ```

   默认情况下，`pr_verify` 仅报告首个不匹配项。若希望查看所有不匹配项，可使用 `-full_check` 选项。
   
2. 关闭当前项目。

   ```tcl
   close_project
   ```

验证通过后，即可安全地为这些配置生成比特流文件。以下是生成比特流文件的具体步骤：

1. 将第一个配置的检查点读入内存。

   ```tcl
   open_checkpoint Implement/Config_shift_right_count_up_implement/top_route_design.dcp
   ```

2. 生成第一个配置的完整比特流和部分比特流。

   ```tcl
   write_bitstream -force -file Bitstreams/Config_RightUp.bit
   close_project
   ```

   此操作将生成以下三个比特流文件：

   - `Config_RightUp.bit`：用于 FPGA 上电时加载的完整比特流；
   - `Config_RightUp_Pblock_inst_shift_partial.bit`：对应 `shift_right` 模块的部分比特流；
   - `Config_RightUp_Pblock_inst_count_partial.bit`：对应 `count_up` 模块的部分比特流。

   若希望自定义部分比特流的文件名，可使用 `-file` 选项设置基本名称，Vivado 会自动添加 RP 的 Pblock 名称作为后缀。在 `advanced_settings.tcl` 脚本中即可进行相关的配置。

3. 生成第二个设计的完整比特流与部分比特流。

   ```tcl
   open_checkpoint Implement/Config_shift_left_count_down_import/top_route_design.dcp
   write_bitstream -force -file Bitstreams/Config_LeftDown.bit
   close_project
   ```

   类似地，此步骤生成了三个比特流，只是拥有不同的名称。

4. 生成带有灰盒（Grey Box）的完整比特流文件，以及可重构区域内对应的空白比特流文件。Vivado 允许为 RP 生成一个“空载”配置，即该配置下不包含任何实际功能逻辑，仅保留可重构边界与结构信息。这类比特流用于擦除当前模块，常用于降低功耗或系统复位。

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

   在“灰盒”配置中，所有 RP 内部逻辑被清空，但输出引脚通过插入 LUT 连接至固定常量（接地），避免悬空连接或非法状态。相比之下，“黑盒”配置则完全移除模块，留下不完整连接，在某些使用场景下可能会影响下游逻辑状态。



### 10. 部分重构 FPGA

在成功生成完整和部分比特流后，我们可以使用 Vivado Hardware Manager 对实际硬件进行编程测试，以验证部分重构功能的正确性。

##### 加载初始配置

1. 打开 Vivado 的 Hardware Manager；
2. 连接 FPGA 设备；
3. 在菜单中选择 *Program Device*；
4. 加载 `Bitstreams/Config_RightUp.bit` 完整比特流；
5. 点击 OK，完成设备编程。

##### 加载部分比特流以进行动态重构

1. 选择 *Program Device*；

2. 依次加载以下部分比特流文件，分别对 `inst_shift` 与 `inst_count` 模块进行替换：

   - `Bitstreams/Config_LeftDown_pblock_inst_shift_partial.bit`

   - `Bitstreams/Config_LeftDown_pblock_inst_count_partial.bit`

每次烧写仅重配置对应的 RP，其余静态逻辑保持运行状态不变。可以观察到：原先向右移动的 LED 列现在向左移动；原本递增计数的指示灯变为递减；所有变化均发生于运行时，无需 FPGA 重启或全片重新编程。
