# SCM

## 相关产品

## 介质

[XL](https://about.kioxia.com/en-jp/news/2019/20190806-1.html)

**Key Features**

- **128 gigabit (Gb) die** (in a 2-die, 4-die, 8-die package)
- **4KB page size** for more efficient operating system reads and writes
- **16-plane architecture** for more efficient parallelism
- **Fast page read and program times.** XL-FLASH™ provides a low read latency of less than 5 microseconds, approximately 10 times faster than existing TLC

[ISSCC XL](https://ieeexplore.ieee.org/document/9063154)

![img](https://ieeexplore.ieee.org/mediastore_new/IEEE/content/media/9046640/9062887/9063154/106_2020_D13_05-fig-5-source-large.gif)

[XL-SSD](https://mp.weixin.qq.com/s/A0_XtMW8pWBsm7lKMh34qQ)

![image-20210827152509767](C:\Users\lee\AppData\Roaming\Typora\typora-user-images\image-20210827152509767.png)

1. TB = 1,000, 000, 000,000Byte.

2. MB/s = 1,000,000 Bytes per second.性能是使用FIO工具在LINUX*Cent 7.5系统上用单线程256队列深度 128KB（131,072 bytes）传输大小的顺序读和写带宽进行测试。*
3. 性能是使用FIO工具在LINUX*Cent 7.5系统上用4线程64队列深度 64*4KB（4096 bytes）传输大小的随机读和写IOPS进行测试。
4. 典型电源消耗是用4线程64队列深度 4096bytes传输大小的随机读/写比例（7/3）进行测试；最大电源消耗是用1线程128队列深度 131,072 bytes 传输大小的顺序写进行测试。

5. μs = =微秒。性能是使用FIO工具在LINUX*Cent 7.5系统上用1线程1QD 4KB（4096 bytes）传输大小的随机/顺序读和写的平均时延进行测试。

6. DWPD: Drive Writes Per Day, last for 5 years.驱动器每天全盘写入次数，持续5年。
7. 该延迟值由Linux Fio libaio io engine测试得到。
8. 该延迟值由Linux SPDK 18.01测试得到。

## 本地测试

![image-20210827153635750](C:\Users\lee\AppData\Roaming\Typora\typora-user-images\image-20210827153635750.png)

## 解决方案

## ==18'OSDI== FLASHSHARE: Punching Through Server Storage Stack from Kernel to Firmware for Ultra-Low Latency SSDs

## ==19'HS== Towards an Unwritten Contract of Intel Optane SSD

 ***Intel Optane 905P VS Samsung 970 Pro***

<img src="C:\Users\lee\AppData\Roaming\Typora\typora-user-images\image-20210827145047966.png" alt="image-20210827145047966" style="zoom: 80%;" />

### Immediate performance: (6)

+ Access with Low Request Scale Rule
+ Random Access is OK Rule
+ Avoid Crowded Accesses Rule
+ Control Overall Load Rule
+ Avoid Tiny Accesses Rule
+ Issue 4KB Aligned Requests Rule

### Sustainable performance: (1)

+ Forget Garbage Collection Rule

### 读写性能测试

<img src="C:\Users\lee\AppData\Roaming\Typora\typora-user-images\image-20210827145142103.png" alt="image-20210827145142103" style="zoom: 67%;" />

+ Rule 1:Access with Low Request Scale

  <img src="C:\Users\lee\AppData\Roaming\Typora\typora-user-images\image-20210827145811976.png" alt="image-20210827145811976" style="zoom: 67%;" />

+ Rule 4: Control Overall Load

## ==19'DAMON== **Exploiting Intel Optane SSD for Microsoft SQL Server**

### Motivation

<img src="C:\Users\lee\AppData\Roaming\Typora\typora-user-images\image-20210827150445454.png" alt="image-20210827150445454"  />

### Design

<table>
    <tr>
        <td><img src="C:\Users\lee\AppData\Roaming\Typora\typora-user-images\image-20210827145142103.png" alt="image-20210827145142103" style="zoom: 67%;" />
        </td>
        <td><img src="C:\Users\lee\AppData\Roaming\Typora\typora-user-images\image-20210827150710579.png" alt="image-20210827150710579" style="zoom:80%;" />
        </td>
    </tr>
</table>

+ **Cache Replacement Policy**

  combine Optane-aware features of each request into the base replacement algorithm (e.g., LRU). 

+ **Cache Access Filter**

  reduce burst loads to the Optane. 

### Conclusion

We present the problems when naively using Optane SSD as a caching layer for Microsoft SQL Server. We provide insights into the problems and propose an Optane SSD-aware caching strategy. We are working on the implementation of this strategy.
