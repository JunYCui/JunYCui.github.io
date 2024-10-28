---
title: Write-Verify Scheme for IGZO DRAM in Analog in-Memory Computing 
date: 2024-10-26 18:43:30 
categories: [Paper, CIM]
tags: [analog, edram]     # TAG names should always be lowercase
math: true
--- 


## 介绍

尽管在深度神经网络（DNN）权重的变化是可以容忍的，但过大的变化（>10%）会降低 DNN 的精度。  
本文针对 IGZO-DRAM 读出设备的**阈值电压变化大**的问题（即权重变化大），提出了一种写入验证的补偿方案，在第一次写入后，将变化的Vth通过ADC读取转化成数字量并与参考值做对比，进而更新权重。

## 写入验证方案 

![写入验证方案](../assets/img/paper/Write-Verify%20Scheme%20for%20IGZO%20DRAM%20in%20Analog%20in-Memory%20Computing/图1.png "写入验证方案")

通过act线激活DRAM，产生ION电流通过一个I\V转换电路，将电流转化为电压，通过ADC量化与init_code作比较通过逻辑电路控制可编程放大器（PGA）控制位线补偿。 


电流转换电压公式
$$
V_{OUT} = I_{ON}(V_{w_{i}}, \delta V^{i}_{TH})*\frac{t_{p}}{C{f}}
$$

$t_{p}$ 表示激活时间，$C_{f}$表示反馈电容

$$
t_{array} = N*t_{cell}
$$
$N$表示每一行的cell数量

## 实验结果

![实验结果](../assets/img/paper/Write-Verify%20Scheme%20for%20IGZO%20DRAM%20in%20Analog%20in-Memory%20Computing/图2.png "实验结果")

从图1中可以看出，$t_{1}$时刻，还未补偿时，$I_{ON}$的方差大，其$\frac{\sigma}{\mu}\approx\pm 27\%$，补偿后(即$t_{2}$时刻)，$\frac{\sigma}{\mu}\approx\pm 3\%$。  
图2展示了$t_{2}$时刻，电流的具体分布
![alt text](image.png)