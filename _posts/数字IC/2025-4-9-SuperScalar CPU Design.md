---
title: SuperScalar CPU Design
author: JYC
description: Note about <<超标量处理器设计>>
date: 2025-4-8 19:12:30 
categories: [Digital Circuit, CPU]
tags: [CPU, SuperScalar]     # TAG names should always be lowercase
--- 

# 寄存器重命名技术

## 概述

一个程序的指令间存在许多的相关性，所谓相关性，是指一条指令依赖另一条指令，从行为上看是指令间的源操作寄存器和目的寄存器相同，一般的数据相关性包含以下三种： 

-  WAW （Write After Write）相关性，两条指令的目的寄存器相同
-  WAR  (Write After Read) 相关性，表示前一条指令的源寄存器与后一条指令的目的寄存器相同
-  RAW  (Read After Write) 相关性，表示前一条指令的目的寄存器与后一条指令的源寄存器相同

相关性又分为假相关和真相关，假相关可以看出只是寄存器的名称相同，但并没有真的依赖关系， WAW 和 WAR 就是典型的假相关，而RAW则是真相关，因为前一个指令的结果真正影响着后一条指令。一个很明显解决假相关问题的方法就是增加寄存器的数量(见下图)，这样就不会有名称的重复，但是对于一个指令集，增加寄存器的数量会导致对以前程序的不兼容，所以提出了寄存器重命名技术，在硬件层面上增加物理寄存器，不增加逻辑寄存器的数量，并通过寄存器重命名技术来实现物理寄存器和逻辑寄存器的映射。

- 物理寄存器指CPU中实际有的寄存器
- 逻辑寄存器指指令中存在的寄存器

![register rename](../assets/img/Superscalar/Register%20Rename.png){: width="972" height="589"}

## 存器重命名

寄存器重命名的任务：读取源操作寄存器、分配目的寄存器和寄存器更新。具体操作如下图，首先通过逻辑寄存器（Arithmetic Register File ,ARF）进行索引物理寄存器（Physical Register File, PRF）索引，然后再通过PRF索引得到真正的寄存器值

![RRE](../assets/img/Superscalar/RRE.png){: width="972" height="589"}

### 重命名映射表

在寄存器重命名中主要运用了重命名映射表，这个表格在具体实现上有两种方式，一种是基于SRAM(SRAM-Based RAT,sRAT)，二是基于CAM(CAM-Based RAT, cRAT)，两种方法各有优劣，下面逐一介绍。 

#### 基于SRAM的重命名映射表

sRAT是指使用SRAM作为表格的载体，其具体形式如下图所示，在图中逻辑寄存器有32个，物理寄存器有64个,固表项(entry)的位宽为5,每个表项内容的位宽为6，所以整个sRAT所占的存储空间为32*6=192bits。

> 值得注意的是因为每条指令都涉及多个寄存器，所以一个周期会有多个寄存器寻址他，所以这是一个多端口SRAM 
{: .box-tip }

![sRAT](../assets/img/Superscalar/sRAT.png){: width="972" height="589"}

#### 基于CAM的重命名映射表

cRAT的具体实现形式如左图所示，其也是**逻辑寄存器进行寻址**，不同的是CAM是一种通过表格内容进行选址的寄存器，可以看出他将逻辑寄存器作为了表格的内容，物理寄存器作为了表项，所以cRAT的存储空间为64*5=320bits显著大于sRAT。

![cRAT](../assets/img/Superscalar/cRAT.png){: width="972" height="589"}

#### cRAT与sRAT的比较

因为SRAM相较于CAM有着更快的访问速度，且基于SRAM的重命名映射表更节省资源，所以sRAT相较于cRAT有着更低的功耗，更小的面积，更快的访问速度。但是目前仍有一些处理器使用着cRAT，因为通过cRAT可以对于每个物理寄存器的状态进行保存，随着Checkpoints的增加，可以cRAT只需要保存每个Checkpoint下物理寄存器的状态就行，sRAT则需要每个checkpoint下的副本而面积显著提升，所以对于一些并行度高，流水线级数深的处理器，cRAT仍然是一个不错的选择。

> CheckPoint 指将某一个时间点寄存器的状态进行保存，类似于git中的快照，由于分支预测失败或着异常等可能的情况，所以处于流水线中的指令都处于推测状态，当出现突发状态要对于sRAT进行恢复并清除处理流水线中的指令，
{: .box-tip }