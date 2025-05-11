---
title: fpga-dfx-abstract-shell
date: 2024-05-29 16:32:36
tags:
typora-root-url: fpga-dfx-abstract-shell
---

# FPGA DFX Abstract Shell

在 Vivado 的 DFX 设计流程中，通常使用上下文编译方法来运行多个配置，这种方法要求在布局布线阶段加载完整的顶层设计。其基本流程为：首先，Vivado 会运行一个包含静态设计部分与动态重构部分的顶层设计的实现，生成第一个已布线的设计检查点（Design Check Point，DCP）。然后，Vivado 会提取 DCP 中的静态设计部分，将其中包含的网表、布局布线信息进行锁定，生成静态设计 DCP，并将其加载到 Vivado 的数据库中。后续配置的实现，都是基于这个静态设计 DCP 构成的上下文启动的。

然而，每次编译一个可重构模块（Reconfigurable Module，RM）都需要完整地加载这个静态设计 DCP，这可能会带来一些限制：

- 若静态设计规模较大，其 DCP 文件体积可能非常庞大，加载进内存耗时明显；
- 静态区域可能包含保密模块或商用 IP，不适合在团队之间或第三方之间共享完整设计；
- 不同 RM 的设计团队必须等待静态设计 DCP 发布，阻碍开发解耦与并行工作；
- ...

为解决上述限制，Vivado 提供了 Abstract Shell 方法。Abstract Shell 为每个 RM 提供了一个抽象的外部 Shell 设计，相当于一个精简的静态设计 DCP，其中只包括与 RM 编译相关的必要逻辑，确保能让 RM 通过端口一致性、功能一致性、时序分析等检测，从而能让各个 RM 进行独立的编译。

使用 Abstract Shell 只需要在原本的 DFX 流程中进行部分修改，其他的部分与原流程保持一致。这里我们以 DFX 的脚本流程作为示例。

以下是默认的 DFX 脚本流程：

```tcl
open_checkpoint config1_routed.dcp
update_design -cell rp1 -black_box
lock_design -level routing
write_checkpoint static_routed.dcp
read_checkpoint -cell rp1 rp1_b_synth.dcp
opt_design
place_design
route_design
write_checkpoint ./rp1_b_routed.dcp
```

如果需要使用 Abstract Shell，则在第一行命令后，直接使用 `write_abstract_shell` 命令该配置生成 Abstract Shell，其流程如下：

```tcl
open_checkpoint config1_routed.dcp
write_abstract_shell -cell rp1 static_ab_routed.dcp
read_checkpoint -cell rp1 rp1_b_synth.dcp
opt_design
place_design
route_design
write_checkpoint ./ab_rp1_b_routed.dcp
```

其中， `write_abstract_shell` 命令会自动执行以下操作：

- 使用 `update_design -black_box` 命令将 DCP 中的可重构区域（Reconfigurable Partition，RP）中的模块替换为黑盒（Black Box）；
- 锁定剩余的设计（包括其他 RM）；
- 为目标 RP 创建 Abstract Shell；
- 使用 `pr_verify` 命令将生成的  Abstract Shell DCP 与原本的 DCP 进行比较，验证上下文一致性。

虽然使用 Abstrac Shell 方法的运行时间比传统提取静态设计上下文的方法更长，但对于大型设计而言，这种方法可以大幅减少后续多个 RM 编译所需时间，有效提升编译效率。

