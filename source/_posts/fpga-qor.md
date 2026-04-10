---
title: fpga-qor
date: 2026-04-10 08:46:23
tags:
typora-root-url: ./fpga-qor
---

# 在 Vivado 中使用 RQA 与 RQS 帮助时序收敛

当设计无法达到目标性能时，可以借助 Vivado 提供的 Report QoR Assessment（RQA）与 Report QoR Suggestions（RQS）进行分析，并据此选择后续的优化方向。RQA 用于评估当前设计的 QoR 状态，RQS 则在此基础上给出可执行的优化建议。二者配合使用，可以帮助我们更高效地推进时序收敛。



### Report QoR Assessment

RQA 用于评估当前设计，并给出一个评分，用于预测设计满足性能目标的可能性。评分含义如下：

| **评分** | **含义**                             |
| -------- | ------------------------------------ |
| 1        | 设计将可能无法完成实现。             |
| 2        | 设计将能够完成实现，但无法满足时序。 |
| 3        | 设计将可能无法满足时序。             |
| 4        | 设计将可能满足时序。                 |
| 5        | 设计将能够满足时序。                 |

RQA 运行得越早，越有机会节省编译时间，因此潜在收益更大；但此时预测精度相对较低，通常误差不超过 1 分。相反，RQA 运行得越晚，评估结果会更准确，但可节省的编译时间也会减少。

不同评分通常对应不同的后续处理方式。常见做法如下：

| **评分** | **常见优化操作**                                             |
| -------- | ------------------------------------------------------------ |
| 1        | 重新设计 HLS 模块。复查设计特性。复查目标器件/速度等级       |
| 2        | 复查约束条件。复查 HLS/RTL 模块。                            |
| 3        | 复查时钟和设计。使用 RQS 运行 ML 策略。使用增量编译达成最终阶段时序收敛。 |
| 4        | 使用 RQS 运行 ML 策略。使用增量编译达成最终阶段时序收敛。    |
| 5        | 运行实现。                                                   |

需要说明的是，应用 RQS 提供的建议后，RQA 评分有可能提高，但这并不是必然结果，仍然取决于设计本身。若设计当前评分为 2，或者处于较低的 3 分，仅依靠 RQS 往往还不够，通常还需要进一步优化 HLS 模块、修改 HDL 代码，或调整 IP 配置。不过在大多数情况下，RQS 的建议仍然具有积极作用，例如改善 WNS 或缓解拥塞。

RQA 还会给出流程引导，帮助用户判断下一步应该采取什么操作。在 Vivado 中运行 Report QoR Assessment 后，可以看到生成的 RQA 报告，如下图所示。

![](1.png)

报告的第一部分是整体评估摘要表，其中给出了 RQA 得分以及对应的流程引导（Flow Guidance）。常见的流程建议主要包括以下几类：是否存在尚未解决的设计方法违规、是否适合采用机器学习策略、是否适合采用增量编译策略，以及是否应进一步应用 RQS 建议。在本例中，设计的评分为 3，说明其达到目标的可能性较低。RQA 给出的建议是运行 RQS，并重点关注报告中标记为 REVIEW 的项目。

第二部分是 QoR 评估详情表，其中汇总了资源利用率与时序相关的评估结果。当某项指标超过阈值时，该条目会被标记为 REVIEW。这里的阈值并不是硬性限制，而是经验参考值，用于提示哪些因素可能导致 QoR 下降。如果某一项指标仅略微超过阈值，通常影响有限；但如果多项指标同时轻微超标，或者某一项显著超标，则设计往往会出现明显问题。因此，这一部分可以作为快速概览工具，用于观察设计当前的风险点，以及应用 QoR 建议前后的变化情况。

第三部分是设计方法检查细节表，用于展示设计中是否存在相关违例。

第四部分是机器学习策略的可用性检查。如果希望使用机器学习策略，一般需要满足以下条件：

- 实现流程已经完成，并执行过 `opt_design`、`place_design`、`phys_opt_design` 和 `route_design`。
- 设计运行时，各阶段 directive 全部使用 Default，或者全部使用 Explore。
- 设计已经发生过关键修改；如果设计与 ML 策略不兼容，并且满足前述条件，应先运行 RQS 以识别问题。
- 目标器件属于 UltraScale 或 UltraScale+ 系列。



### Report QoR Suggestions

RQS 用于根据当前设计的 QoR 情况，给出改善时序性能的建议，包括可采用的命令、属性以及实现策略。在 Vivado 中运行 Report QoR Suggestions 后，可以看到生成的 RQS 报告，如下图所示。

![](2.png)

报告的第一部分是 QoR 建议摘要表，其中列出了每一条 RQS 建议。建议通常分为六类：Utilization、XDC、Usage、Congestion、Timing 和 Strategy。在本例中，RQS 给出了两条时序相关建议。第一条建议指出，设计中存在负载分布较远的网络，因此可以在综合完成后，对关键网络施加 `FORCE_MAX_FANOUT` 属性，通过复制驱动器改善时序。第二条建议指出，某些 setup 关键路径可以通过复制 LUT 驱动网络来进一步优化。

第二部分给出了机器学习策略，对 `opt_design`、`place_design`、`phys_opt_design` 和 `route_design` 四个阶段分别推荐了可使用的 directive。本例中共提供了三种策略。

第三部分是时序优化建议，展示了被关注的目标路径及其详细信息，便于进一步定位问题。



### 案例研究

下面通过一个简单案例，说明如何结合 RQA 与 RQS 推进时序收敛。RQA 用于给出设计评分和流程指引，RQS 用于提供具体的优化建议和实现策略。

这里以一个 logicnets_jscl 检查点为例，如下图所示。

![](3.png)

该设计已经完成布线，目标频率为 666.7 MHz。根据时序报告，布线结束后的结果为：WNS = -0.978 ns，Failing Endpoints 数量为 1529。

接下来，我们在非工程模式下使用 RQA 与 RQS 对该设计进行优化。首先，在 Vivado 中选择 `Report -> Report QoR Assessment` 生成 RQA 报告。随后，根据 RQA 给出的流程建议，继续选择 `Report -> Report QoR Suggestions` 生成 RQS 报告。得到建议后，就可以按照 RQS 提供的策略执行后续优化。整体流程如下图所示。

![](4.png)

当 `report_qor_suggestions` 生成完成后，可以在 GUI 中或 Tcl Console 中使用 `write_qor_suggestions` 命令导出所选建议。例如：

```bash
write_qor_suggestions -of_objects [get_qor_suggestions {RQS_TIMING-3-1 RQS_TIMING-59-1 RQS_STRAT-2-1 RQS_STRAT-35-1 RQS_STRAT-39-1 }] -file /home/wenbinteng/xilinx/rqs_report.rqs -strategy_dir /home/wenbinteng/xilinx/MLStrategy -force
```

在这里，`-strategy_dir` 用于指定保存机器学习策略及其配套脚本的目录。导出后，可以在后续流程中直接应用这些策略。对应的非工程模式 Tcl 脚本如下：

```bash
set RQSFile "/home/wenbinteng/xilinx/MLStrategy/checkpoint_logicnets_jsclSuggestionFile1.rqs"
read_qor_suggestions $RQSFile
opt_design -directive RQS
place_design -directive RQS
phys_opt_design -directive RQS
route_design -directive RQS
```

在 Tcl 终端执行上述命令后，Vivado 会按照 RQS 建议重新完成实现流程。优化结束后再次查看时序报告，可以看到结果变为：WNS = -0.931 ns，Failing Endpoints 数量为 1528。说明该轮优化带来了一定改善，但距离最终收敛仍有差距，后续还需要继续结合设计分析进行迭代优化。



### 参考文献

[1] https://docs.amd.com/r/en-US/ug906-vivado-design-analysis

[2] https://adaptivesupport.amd.com/s/article/1110761?language=en_US

[3] https://adaptivesupport.amd.com/s/article/1033308?language=en_US

[4] https://adaptivesupport.amd.com/s/article/1118594?language=en_US
