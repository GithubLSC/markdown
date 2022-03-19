# ==21‘OSDI== Rearchitecting Linux Storage Stack for *µ*s Latency and High Throughput[^1]



## Abstract

​		作者团队针对 支持多队列存储系统 做了内核存储栈的优化。在多应用运行在一个CPU上的情况下，方案考虑了各个进程的类别，充分利用 存储设备硬件多队列机制 与 闲置的CPU资源，做了选择性调度，负载均衡，同时保证了延迟敏感型应用的低延迟和带宽敏感型应用的高吞吐量。

<img src="C:\Users\lee\AppData\Roaming\Typora\typora-user-images\image-20210716194058278.png" alt="image-20210716194058278" style="zoom:67%;" />

## Introduction

+ 传统多队列机制没有考虑应用的类别，可能会导致HOL Blocking 队头阻塞问题 [^2]，当多个进程运行在同一个CPU时，存储系统无法保障低延迟和高带宽。

<img src="C:\Users\lee\AppData\Roaming\Typora\typora-user-images\image-20210716194819535.png" alt="image-20210716194819535"  width="800px"  height="250px" style="zoom:80%;" />

+ 下图清楚地展示了Linux 原生多队列机制可能引发的的HOL现象，isolated表示只有一个LAPP在运行，1，2，4 则表示一个TAPP运行的前提下，不断增加LAPP的数量，统计L类型的APP的延迟。

<img src="C:\Users\lee\AppData\Roaming\Typora\typora-user-images\image-20210718104442202.png" alt="image-20210718104442202" style="zoom:80%;" />

## Background

### Linux

 <table>
      <tr>
          <td>HOL blocking导致LAPP高延迟问题。</td>
          <td><img src="C:\Users\lee\AppData\Roaming\Typora\typora-user-images\image-20210716212647343.png" alt="image-20210716212647343" style="zoom:60%;" /></td>
      </tr>
  </table>
### SPDK

> SPDK的目标是通过使用Intel的网络，处理，存储技术，将固态存储介质出色的功效发挥到极致。相对于传统IO方式，SPDK运用了两项关键技术：UIO和pooling。
>
> + 将设备驱动代码运行在用户态，避免内核上下文切换和中断将会节省大量的处理开销，允许更多的时钟周期被用来做实际的数据存储。
> + 采用轮询模式改变了传统I/O的基本模型。

 <table>
      <tr>
          <td>延迟和吞吐量都无法保证。</td>
          <td><img src="C:\Users\lee\AppData\Roaming\Typora\typora-user-images\image-20210716212704551.png" alt="image-20210716212704551" style="zoom:70%;" /></td>
      </tr>
  </table>

### SPDK+Priority

 <table>
      <tr>
          <td>导致带宽敏感型应用饥饿。</td>
          <td><img src="C:\Users\lee\AppData\Roaming\Typora\typora-user-images\image-20210716212722647.png" alt="image-20210716212722647" style="zoom:70%;" /></td>
      </tr>
  </table>

<table>
    <tr>
        <td><img src="C:\Users\lee\AppData\Roaming\Typora\typora-user-images\image-20210716201614283.png" alt="image-20210716201614283" style="zoom:80%;" /></td>
        <td><img src="C:\Users\lee\AppData\Roaming\Typora\typora-user-images\image-20210716202203123.png" alt="image-20210716202203123" style="zoom:80%;" width="400" height=250/></td>
    </tr>
</table>
## Design

### 方案

<font color="red">解耦：将CPU上的软件队列IO请求拆解分类，利用多硬件队列，采用优先级策略，多CPU负载均衡。</font>

+ blk-mq软件队列与设备硬件队列不再是一一对应，而是同一个核的软件队列可以映射到多个硬件队列上 。
+ 为了更好的服务延迟  敏感型应用，赋予处理LAPPio请求的线程更高的优先级。
+ 调度
  + request级别调度：为了避免T类型应用饥饿。将本软件队列中的T类型请求，映射到空闲CPU的硬件队列上。
  + Application级别调度：为了实现负载均衡，同时减少同一个核的LAPP和TAPP互相调度切换开销，将异类型APP分配到空闲的CPU运行。
+ 优点
  + T类型APP保证了高吞吐量
  + 由于减少了同一个核的调度时间，更少的上下文开销保证了L类型应用的低延迟。

![image-20210714104950371](C:\Users\lee\AppData\Roaming\Typora\typora-user-images\image-20210714104950371.png)

### 实现细节

+ Multi-Egress
+ Request Steering
+ Application Steering

<table>
    <tr>
        <td> <img src="C:\Users\lee\AppData\Roaming\Typora\typora-user-images\image-20210716213529333.png" alt="image-20210716213529333" style="zoom: 110%;" /><font color="green"><center><b>Multi-Egree</b></center></font></td>
        <td><img src="C:\Users\lee\AppData\Roaming\Typora\typora-user-images\image-20210716213551793.png" alt="image-20210716213551793" style="zoom: 120%;" /><font color="green"><center><b>Steering</b></center></font></td>
    </tr>
</table>

## Evaluation

### 实验准备

+ 作者团队 主要利用远程内存作为块设备 借用i10存储栈优化以及100Gbps高速网络，模拟数据中心高速通信场景。

  文章题目是us级别延迟，而本地的nvme设备基本的读延迟为80us( *Samsung PM1735 MLC*)，一定程度上掩盖了blkswitch的性能优势，因此实际盘远程测试只包含一组实验。

<img src="C:\Users\lee\AppData\Roaming\Typora\typora-user-images\image-20210716210757637.png" alt="image-20210716210757637" style="zoom:60%;" />

### 实验结果

+ 指标
  + 延迟
  + 吞吐量
  
+ 压力测试

  一个T类型APP ，改变L类型APP的个数，统计LAPP的延迟和CPU吞吐量。

  <img src="C:\Users\lee\AppData\Roaming\Typora\typora-user-images\image-20210718111415388.png" alt="image-20210718111415388" style="zoom:80%;" />

+ 性能优势分解实验 Performance breakdown

  + Baseline

  + Baseline+Priority

  + Baseline+Priority+Request Steering

  + Baseline+Priority+Request Steering+App Steering

  
  <img src="C:\Users\lee\AppData\Roaming\Typora\typora-user-images\image-20210716211425206.png" alt="image-20210716211425206" style="zoom: 33%;" />
## Conslusion 

  + 在Linux上 同时实现us级别延迟和高吞吐量是可行的。

## 本地实验

### 基本情况

+ 缺少100Gbps以太网，远程内存块设备访问实验无法开展。

+ 本地测试无效果。


### 分析

#### 本地访问为什么没有低延迟？

+ 文章在决定Steering时设定了许多阈值，这些阈值的设置可能与实验环境有关。


  + 文章实现了10us级别延迟， 可以推算出优化后blk层的开销可能在也在us级别，而文章给出本地SSD的基本访问延迟在100us左右。可能在当前本地nvme设备下，对blk层的优化并无太大实际价值，BIO层并非存储系统瓶颈。
    + i10即使做了io栈的优化，但是实验室本地复现无效果。
    + 预取的Leap也是基于数据中心RDMA高速网络（4.3us），io栈的开销成为了瓶颈问题，其实用在本地盘上，效果也不大。

#### Blkswitch 适用场景？

+ 数据中心高速网络 

  文章的 motivation HOL Blocking现象，是网络领域的经典问题，恰巧存储栈也面临类似的问题。

+ 数据存储中心 + 多租户场景

  + blkswitch 最大的收益点，在利用闲置的硬件队列，闲置的CPU解决在多用户访问场景。但方案有效的前提是，不同用户有不同的CPU资源的使用率。各不相同。

  + 在==储存服务器==场景下，充分利用现有CPU，获得最佳性能 (Latency Throughput) 是目标。
  + 而在==本地场景==下，现有工作最大程度上降低CPU的开销，让存储系统以最小的CPU开销实现高性能，CPU主要用来解决计算问题。

[^1]:https://www.usenix.org/conference/osdi21/presentation/hwang
[^2]:<img src="https://upload-images.jianshu.io/upload_images/6383319-0038959f76865b38.jpg?imageMogr2/auto-orient/strip|imageView2/2/format/webp" alt="img" style="zoom: 80%;" />
