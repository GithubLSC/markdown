# SWAP设计原则

# SWAP解决方案

### ==21'ATC== Fast mobile Application Switch via Adaptive Prepaging

+ SWAP IN ，有点文件级别预取的思想。（涉及文件页面和匿名页）
+ Reducing switching latency by leveraging prepaging
+ 文章是首尔大学，成均馆大学，谷歌的工作主要解决移动设备应用切换延迟问题。
+ 作者首先介绍了android 内存回收机制swap 和lmkd，分析手机应用切换高延迟的成因。
+ 随后作者通过一系列实验说明demanding pagefault 是先当耗费时间的，同时观察到当前page fault时cpu和磁盘带宽的利用率都不高。由此，作者指出，采用prepaging 策略可以充分利用应用切换过程的CPU资源和存储资源，改善应用切换时启动慢的问题。
+ Prepaging 需要确定两个问题
  1. 取哪些页面 作者记录页面切换过程访问的页面，发现大多数页面的固定的（分为匿名页和文件页）。
  2. 怎么取 批量取，并且保证取页面的线程不会占用过多的CPU资源和应用线程竞争

### ==20'TCAD== SEAL :User Experience-Aware Two-level Swap for mobile devices

+ SWAP IN 和 SWAP OUT，批处理，ZRAM 和 swapfile 两级swap（只涉及匿名页，不涉及文件页面）

+ 文章新提出了一个swap观测指标 ：帧率。

+ 文章介绍了swap的背景，冷启动和热启动，zram和swapfile 并指出，两种机制有相应的优缺点。zram速度快，但由于受到内存容量的限制，可缓存的最大应用数少，而存储swap机制，扩大了交换分区，但是io较慢。因此，作者指出将两种机制结合利用，两级swap。

  <img src="C:\Users\lee\AppData\Roaming\Typora\typora-user-images\image-20210907155149079.png" alt="image-20210907155149079" style="zoom:67%;" />

+ 另外，作者创新性提出，应用在启动阶段和运行阶段，所需要的swap文件，是不同的。由于启动所需要的文件一般是固定的，而用户对启动延迟的容忍度较低，因此应该将启动所需的文件放入一级SWAP zram中，而运行阶段所需的文件放入二级swap swapfile 。

  <img src="C:\Users\lee\AppData\Roaming\Typora\typora-user-images\image-20210907155113623.png" alt="image-20210907155113623" style="zoom:67%;" />

+ 另外，为了避免由于部分swap文件本来可以放置在一级swap，但为了缓存应用数增加，却放在二级swap，文章提出了隐藏页面加载时延(从二级swap预加载到一级swap）的策略。根据应用启动延迟，计算出能放到二级swap的页面数量，并在应用启动的同时，后台预加载页面到一级swap。

![image-20210907155020829](C:\Users\lee\AppData\Roaming\Typora\typora-user-images\image-20210907155020829.png)

+ 此外，方案在进行swap时，采用了更为aggressive的策略，首先，在一级swap 到 二级swap的同时，采用批量迁移的策略，其次，在迁移时，当属于应用的页面迁移结束后才停止，在内存资源充足的情况下，迁移并不会提前停止。

## ==20'ATC== Acclaim: Adaptive Memory Reclaim to improve User Experience in Android Systems

+ SWAP OUT, 后台应用 自适应swap out 窗口，并赋予前台应用高优先级。

+ 文章首先介绍了内存回收机制 kswapd 和 direct reclaim。

  <img src="C:\Users\lee\AppData\Roaming\Typora\typora-user-images\image-20210907142207777.png" alt="image-20210907142207777" style="zoom:80%;" />

+ 然后，作者发现，页面请求通常是小粒度，而直接页面回收则为大粒度，这就导致了在内存资源紧张的情况下，页面会经常发生page refault 即页面需要的页面被交换出内存（或ZRAM），该页面重新访问则引入了开销。

  ![image-20210907142637537](C:\Users\lee\AppData\Roaming\Typora\typora-user-images\image-20210907142637537.png)

+ 文章介绍了应用启动切换延迟问题，并指出两大元凶 page refault 和 direct reclaim。根本原因还是在于页面回收的粒度过大。

+ 此外，作者还指出，后台进程和系统进程仍然会消耗空闲页面。

+ 作者根据 某个进程 一定时间窗口内请求的页面数来决定该进程页面回收的粒度

+ 同时，赋予后台进程更低的优先级，来保证换出时，优先换出后台应用的页面，来保障前台应用的低延迟。

  ![image-20210907144252424](C:\Users\lee\AppData\Roaming\Typora\typora-user-images\image-20210907144252424.png)

## MobiFlash：Expanding Mobile Memory with Flash

+ SWAP IN 和 SWAP OUT ，新存储介质SCM ZNAND

+ 作者介绍了应用的切换延迟问题，并对两种SWAP ，ZRAM和Flash swap的延迟进行了分解。

  ![image-20210907150623833](C:\Users\lee\AppData\Roaming\Typora\typora-user-images\image-20210907150623833.png)

+ 方案同样是通过解决swap带来的启动切换延迟问题，但是利用了ZNAND和ARM cash stashing技术。

+ 三个优化点

  1. 软件栈的优化 (页表对应SWAP直接对应页面LPN，主机端缓存Mapping table)
  2. 硬件优化 TLC->ZNAND

![image-20210907150829348](C:\Users\lee\AppData\Roaming\Typora\typora-user-images\image-20210907150829348.png)

+ 实验

  <table>
      <tr>
      <td><img src="C:\Users\lee\AppData\Roaming\Typora\typora-user-images\image-20210907150912802.png" alt="image-20210907150912802" style="zoom:100%;" />
          </td>
      <td><img src="C:\Users\lee\AppData\Roaming\Typora\typora-user-images\image-20210907150938034.png" alt="image-20210907150938034" style="zoom:100%;" />
          </td></tr>
  </table>
  
