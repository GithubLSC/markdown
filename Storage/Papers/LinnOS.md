# ==20'OSDI==  LinnOS: Predictability on Unpredictable Flash Storage with a Light Neural Network

## Abstract

+ 作者团队通过轻量级的神经网络预测闪存设备的读延迟. 并将模型整合到操作系统内核中,预测单条读请求的延迟,开销小,准确度高.
+ 技术用于解决 分布式系统\RAID 等数据存在多备份的场景下.

## Introduction

+ 由于SSD内部的复杂结构(GC WB WL ECC),其读写性能存在严重的尾端延迟现象.
+ 目前业界解决尾端延迟问题主要从Hedging request[The tail at scale](http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.732.6087&rep=rep1&type=pdf)的角度展开,问题在于加重了分布式系统的数据量.

> *A simple way to curb latency variability is to issue the same request to multiple replicas and use the results from whichever replica responds first.*

+ 三种策略

  + 白盒

    + 需要厂商修改固件
    + 不能掩饰不可预测性
  + 灰盒

    + 文件系统及应用的优化重新设计
    + 需要修改固件
    + 缓解不可预测性
  + 黑盒

    + 掩饰延迟不可预测性

    - Hedging Request

## Background

+ 写请求主要由DRAM WB 缓冲, 少部分尾端延迟不影响用户体验.
+ 读请求经常会受到内部写请求的干扰.
+ 不同的设备长尾延迟出现点不同

![image-20210708154510949](C:\Users\lee\AppData\Roaming\Typora\typora-user-images\image-20210708154510949.png)

+ 内部复杂性

  + io落入同意通道或者芯片，请求互相竞争
  + Hedge request对于粗粒度的任务效果好，但是对于闪存没有效果。当预期的响应时间小于几毫秒时，等待的代价很高。
+ 机器学习

  + 二分类问题。
  + 准确率和性能之间存在Trade-off

## Overview

+ 使用场景: 冗余存储系统

> ***Linnos is beneficail for parallel, redundant storage such as flash arrays (cluster-based or RAID) that maintain multiple replicas of the same block.***

+ 使用案例: 预测当前设备读请求延迟,若是一条快请求,则请求下发,否则,撤销该请求,转发到其它设备上......
+ 模块
  + Model：尽可能减小特征的个数
  + Tracing ：设备负载单独收集
  + Labeling：inflection Point算法
  + Training ：监督学习
  + Uploading weights：支持在线预测

![image-20210708184241061](C:\Users\lee\AppData\Roaming\Typora\typora-user-images\image-20210708184241061.png)

+ 模型工作流程
  + 当存储应用通过系统调用处理一个I/O请求的时候，LinnOS会增加1 bit的flag(latency-critical(LC) = false/true)。
  + 在将I/O提交给底层SSD时，会先将I/O信息输入到训练好的神经网络模型中，然后会得到一个结果（fast/slow）。
  + 如果得到的结果是fast，就直接将I/O提交给device。
  + 如果得到的结果是slow，就revoke I/O，并返回“slow”的错误码。
  + application接收到错误码之后，就会发送一个相同的I/O给另一个replica。
  + 最差的情况是遍历到最后一个replica，但是最后这个replica返回的retry将不设置LC，主要是为了让最后这个replica被处理并不被继续的revoke。

![image-20210708161451296](C:\Users\lee\AppData\Roaming\Typora\typora-user-images\image-20210708161451296.png)

![image-20210708154353943](C:\Users\lee\AppData\Roaming\Typora\typora-user-images\image-20210708154353943.png)

## LinnOS Design

+ 训练数据集

  真实数据量越多，在训练模型时对应的模型准确率会更高，但是训练的数据越多占用的CPU资源会越多。
+ 拐点决定算法

  + 随机找一个包含想要数据的replica对应的SSD，然后运行负载模拟。
  + 更新Inflection Point value的步骤是：the current IO request:
    1）找到斜率是45度的点a，a.x就是Inflection Point value。
    2）当current latency < a.x时，.latency = current latency；当current latency > a.x时，撤销的同时将相同的IO request随机故障转移到包含相同的replica的node上。
    3）分别计算原始的和新的latency围成的CDF面积，比较面积之间的差异。
    4）当初始的Inflection point value在+/-10%范围内波动时移动+/-0.1%。然后重复执行1），2），3）。
  + 找到optimal Inflection Point value用作模型区分slow/fast的阈值。

![image-20210708153637169](C:\Users\lee\AppData\Roaming\Typora\typora-user-images\image-20210708153637169.png)

+ 神经网络模型

  + 输入参数

    1. 等待的IO数量
    2. 最近R个请求的延迟分布
    3. R个请求的等待io数量
  + 输入格式
  + 网络(全连接)

    输入层

    隐藏层

    输出层
+ 准确度提升

  + Reducing false submits: 自定义Hinge loss function
  + recalibrating: 拐点发生偏移时,重新采集trace ,更新权重
  + masking small inaccuracy: 结合Linnos 和Hedging的优点

<img src="C:\Users\lee\AppData\Roaming\Typora\typora-user-images\image-20210708185359232.png" alt="image-20210708185359232" style="zoom:80%;" />

+ 优势
  + 可预测性
  + 自动化
  + 通用性
  + 时间开销小

![image-20210708153550931](C:\Users\lee\AppData\Roaming\Typora\typora-user-images\image-20210708153550931.png)

<img src="C:\Users\lee\AppData\Roaming\Typora\typora-user-images\image-20210708154139545.png" alt="image-20210708154139545" style="zoom: 80%;" />

## Evaluation

+ 负载 设备
+ 其他方案
  + Baseline
  + Clone: 每个io请求发送两次
  + HeruSim: 根据队列深度判断是否要重发请求
  + HeruADV: 跟据最近请求的延迟和当前队列深度 决定是否提交请求
  + Hedge95 : 默认95为拐点
  + HedgeIP : 使用面积确定semi-optimum 拐点
  + LinnOS
  + LinnOS+HL 结合Linnos和Hedging 的优点

![image-20210708153755831](C:\Users\lee\AppData\Roaming\Typora\typora-user-images\image-20210708153755831.png)

![image-20210708154257240](C:\Users\lee\AppData\Roaming\Typora\typora-user-images\image-20210708154257240.png)

## Conclusion and Discussion

- LinnOS，第一个能够推断每个I/O到闪存存储的速度的操作系统地方。
- 作者经证明了在操作系统中使用轻量级神经网络进行频繁、细粒度、黑盒实时推断的可行性。
- LinnOS优于许多其他方法，并成功地为不可预测的闪存存储带来了可预测性。
