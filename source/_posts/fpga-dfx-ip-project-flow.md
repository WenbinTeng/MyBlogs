---
title: fpga-dfx-ip-project-flow
date: 2025-05-09 08:58:17
tags:
typora-root-url: fpga-dfx-ip-project-flow
---

# FPGA DFX IP Project Flow

在大型项目中，常常使用 IP 来模块化地管理设计源代码，以方便地管理系统中各个模块的功能、组织模块之间的互联。Vivado 2021.2 版本更新了基于 IP 管理的项目设计，可以让我们更方便地在模块设计（Block Design，BD）中使用 DFX 功能。基于 IP 项目的 DFX 设计包含以下特性：

- 能够创建模块设计容器（Block Design Containers，BDC），并能在项目结构中识别其层次。
- 将 BDC 定义为可重构分区（Reconfigurable Partitions，RP）。
- 为每个 RP 创建一组可重构模块（Reconfigurable Modules，RM）。
- 创建顶层模块与子模块综合运行、创建配置、创建实现运行、验证配置、生成完整比特流与部分比特流等，与基于 RTL 的项目设计类似。

开始实验之间，我们先从 Xilinx 官网下载官方 DFX 教程设计文件：

[下载链接](https://www.xilinx.com/member/forms/download/design-license.html?cid=03e48cb4-ba89-496d-a3d7-cbaa2302ef79&filename=ug947-vivado-partial-reconfiguration-tutorial.zip)

下载完成后，将文件解压至任意具备写权限的本地路径，然后进入解压后的 `ipi_bdc_dfx_zu ` 子目录。本实验将使用 ZCU102 作为目标开发板，所有步骤都将在该目录下完成。所有编译步骤可以使用 `run_all.tcl` 以 Tcl 脚本的形式运行，这里为了展示 DFX 的功能，将使用 GUI 中的 IP 集成器（IP Integrator）进行演示。



### 1. 在 IP 集成器中创建设计

1. 启动 Vivado IDE。在 Tcl 控制台中，导航到教程存档解压的文件夹 `ipi_bdc_dfx_zu ` 。

2. 激活第一个 Tcl 脚本已创建一个针对 ZCU102 开发板的平面设计。

   ```tcl
   source create_top_bd.tcl
   ```

   这个脚本会执行一些任务：

   - 为 ZCU102 目标开发板创建一个新项目；
   - 添加并自定义一组 IP；
   - 在 BD 中连接各个 IP；
   - 验证并保存 BD 设计。

   这个 ` create_top_bd.tcl` 脚本是从现有的设计中提取的，使用 `write_bd_tcl -no_ip_version` 命令就可以完成这一操作。

   <img src="1.png" style="zoom:80%;" />

<img src="2.png" style="zoom:80%;" />
