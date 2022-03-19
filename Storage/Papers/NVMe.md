# <center>NVMe 文章调研</center>

+ **多核负载均衡**
  
  Setting CPU affinity
  
  + SSDCheck
  + Blkswitch
  
+ **多SQ负载均衡**

  + Blkswitch
  + AS
  + SSDCheck(SQ的饥饿现象)

+ **定制化io服务**
  
  + Blkswitch
  + AS
  + SPDK
  + WRR
  + NVMeDirect

# Background

##  ==13‘Systor== Linux Block IO: Introducing Multi-queue SSD Access on Multi-core Systems

<table>
    <tr>
        <td>
            <img src="C:\Users\lee\AppData\Roaming\Typora\typora-user-images\image-20210724111454884.png" alt="image-20210724111454884" style="zoom:60%;" />
        </td>
        <td>
            <img src="C:\Users\lee\AppData\Roaming\Typora\typora-user-images\image-20210724105929774.png" alt="image-20210724105929774" style="zoom:60%;" />
        </td>
    </tr>
</table>

## ==16‘== **Solving the Linux storage scalability bottlenecks**

![image-20210815184535817](C:\Users\lee\AppData\Roaming\Typora\typora-user-images\image-20210815184535817.png)

# 限流

##  ==21’== FlashDAM: Dynamic I/O Throttling for Performance Improvement in SSDs

= MQSim+LinnOS

+ 背景

  适当限流有助于维持峰值性能

  ![image-20210816090837040](C:\Users\lee\AppData\Roaming\Typora\typora-user-images\image-20210816090837040.png)

+ 问题
  + 读写干扰影响读性能。
  + 顺序随机交织造成Mapping Table缓存污染。
  + 重载情况下，ssd温度升高，cpu降频，影响性能。
  
+ 设计
  + 双队列设计，采集最近8个请求的延迟以及请求的特征。
  + 根据输入特征利用随机森林预测SSD内部的繁忙状态。
    +  I/O depth
    + read/write size
    + sequential degree (SD)
    + alternation degree (AD)
  + 捕获请求的时机与捕获请求的数量 可以动态调整

<img src="C:\Users\lee\AppData\Roaming\Typora\typora-user-images\image-20210811215938538.png" alt="image-20210811215938538" style="zoom:80%;" />

# BLK-MQ 

## ==20‘AS== Practical Enhancement of User Experience in NVMe SSDs

+ 问题：定制化的io服务

<img src="C:\Users\lee\AppData\Roaming\Typora\typora-user-images\image-20210815190637701.png" alt="image-20210815190637701" style="zoom:80%;" />

+ 设计：

  + 单个CPU对应两个软件队列，分为低优先级和高优先级，在派发到硬件队列时，优先选择高优先级软件队列中的请求。
  + 将请求从SWQ取出，加入HWQ队列时，选取SQ命令最少的HWQ，可以得到相对优先的处理

+ 效果：

  将请求队列分为以用户体验为中心和其他，加大了队列管理算法的复杂度，但优化了用户体验。

<table>
    <tr>
    <td>
     <img src="C:\Users\lee\AppData\Roaming\Typora\typora-user-images\image-20210727164047751.png" alt="image-20210727164047751" style="zoom:80%;" /> </td>
        <td>
        <img src="C:\Users\lee\AppData\Roaming\Typora\typora-user-images\image-20210727164101153.png" alt="image-20210727164101153" style="zoom:80%;" /></td>
    </tr>
</table>
## ==17’IMCOM==  Improving Read Performance by Isolating Multiple Queues in NVMe SSDs 

![image-20210903152534326](C:\Users\lee\AppData\Roaming\Typora\typora-user-images\image-20210903152534326.png)

##  ==21‘ToS== (SSDCheck) Performance Modeling and Practical Use Cases for Black-Box SSDs

+ 问题：SQ的并行性与多SQ之间的公平性

+ 设计：充分利用底层的多SQ队列，并且在blkmq层限制软件队列的深度。

  <table>
      <tr>
      <td><img src="C:\Users\lee\AppData\Roaming\Typora\typora-user-images\image-20210815174731270.png" alt="image-20210815174731270" style="zoom:80%;" />
          </td>
      <td><img src="C:\Users\lee\AppData\Roaming\Typora\typora-user-images\image-20210815171240671.png" alt="image-20210815171240671" style="zoom:80%;" />
          </td></tr>
  </table>

  + **MQ-aware thread mapping.**
  + **MQ-aware queue throttling.**

<img src="C:\Users\lee\AppData\Roaming\Typora\typora-user-images\image-20210815165252069.png" alt="image-20210815165252069" style="zoom:80%;" />

+ 效果：提升一个核上运行多个任务的整体性能，并且保证多个核之间的公平性。




![image-20210815174225980](C:\Users\lee\AppData\Roaming\Typora\typora-user-images\image-20210815174225980.png)

## ==21‘OSDI== Rearchitecting Linux Storage Stack for *µ*s Latency and High Throughput

+ 问题：多队列并行性与负载均衡调度

+ 设计：

  + request级别调度：为了避免T类型应用饥饿。将本软件队列中的T类型请求，映射到==空闲CPU==的硬件队列上。
  + Application级别调度：为了实现负载均衡，同时减少同一个核的LAPP和TAPP互相调度切换开销，将异类型APP分配到空闲的CPU运行。

+ 效果：

  配合i10的高速网络优化，实现了延迟敏感型应用的us级别低延迟，同时保证了带宽敏感型应用的吞吐量

![image-20210714104950371](C:\Users\lee\AppData\Roaming\Typora\typora-user-images\image-20210714104950371.png)


# NVMe Driver 

## WRR

NVMe对于使用不同的队列定义了调度策略，用于区分不同优先级的任务。

<img src="http://mmbiz.qpic.cn/mmbiz/B3RBjauFmgiaBydoHEQOx5P1dvAibAFKK5oMTwn9eUe4j6HjWUAcSpvlqsy17MRCpJ9rmiaZuYJ4unw2Caics1tqsw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1" alt="图片" style="zoom:80%;" />

## ==21’OSDI== Optimizing Storage Performance with Calibrated Interrupts

### NVMe coalescsing

![image-20210830152408049](C:\Users\lee\AppData\Roaming\Typora\typora-user-images\image-20210830152408049.png)

### Design

+ urgent
+ barrier

<table>
    <tr>
    <td><img src="C:\Users\lee\AppData\Roaming\Typora\typora-user-images\image-20210830152212948.png" alt="image-20210830152212948" style="zoom:67%;" />
        </td>
    <td><img src="C:\Users\lee\AppData\Roaming\Typora\typora-user-images\image-20210830152139877.png" alt="image-20210830152139877" style="zoom:80%;" />
        </td></tr>
</table>


## ==17’HotStorage== Enabling NVMe WRR support in Linux Block Layer

+ 问题：io服务的定制化设计
+ 设计：借用用CFS调度的四种调度类，每个 cq对应四个sq队列，按照wrr取io command 到设备端

+ 效果：赋予不同类型的进程不同的io请求处理速度。



<table>
    <tr>
        <td><img src="C:\Users\lee\AppData\Roaming\Typora\typora-user-images\image-20210815165533366.png" alt="image-20210815165533366" style="zoom:80%;" />  <font color="green"><center><b>SPDK</b></center></font>
        </td>
        <td><img src="C:\Users\lee\AppData\Roaming\Typora\typora-user-images\image-20210815165429419.png" alt="image-20210815165429419" style="zoom:80%;" /><font color="green"><center><b>WRR</b></center></font>
        </td>
    </tr>
</table>
# User-space NVMe

## SPDK

## ==16’HotStorage== NVMeDirect: A User-space I/O Framework for Application-specific Optimization on NVMe SSDs

SPDK只适用于单个用户和单一应用，因为它把整个NVMe驱动从内核空间移动到了用户空间

+ 问题：面向应用程序的 nvme设备 io栈优化。

  ==内核就应该是通用的，因为它为应用提供了一个抽象层，负责管理所有硬件资源。因此，对内核进行优化而不损失通用性是很困难的。==

+ 设计：

  + 类似SPDK，实现用户态NVMe。
  + 自定义用户库函数API，将创建的队列的内核空间的内存地址和门铃寄存器映射到用户态地址空间。
  + 轮询取代中断。
  + 实现用户态块缓存，io调度器。
  + 用户可以直接管理队列，提供差异化的io服务。单线程独享队列，读写队列分离。

+ 效果

  + 现有的应用和启用了NVMeDirect的应用可以共享同一块NVMe固态硬盘。然而在SPDK中，整块NVMe固态硬盘是专属于某一个过程，该进程拥有所有的用户级别的驱动代码。

<table>
    <tr>
    	<td><img src="C:\Users\lee\AppData\Roaming\Typora\typora-user-images\image-20210727165951008.png" alt="image-20210727165951008" style="zoom:80%;" />
    	</td>
    	<td>
    	    <img src="C:\Users\lee\AppData\Roaming\Typora\typora-user-images\image-20210727165814645.png" alt="image-20210727165814645" style="zoom:80%;" />
    	</td>
    </tr>
</table>

## ==17’Flash Memory Summit== NVMeDirect 2.0: An Enhanced User-space I/O Framework for NVMe SSDs

![image-20210815213509911](C:\Users\lee\AppData\Roaming\Typora\typora-user-images\image-20210815213509911.png)

# SSD Simulator

## ==18‘FAST== **MQSim: A Framework for Enabling Realistic** Studies of Modern Multi-Queue SSD Devices
