# UFS 

## WriterBooster

+ Using SLC NAND as a WriteBooster Buffer enables the write request to be processed with lower latency and improves the overall write performance. 

![image-20210903193322921](C:\Users\lee\AppData\Roaming\Typora\typora-user-images\image-20210903193322921.png)

## Buffer Mode

### LU dedicated buffer mode

![image-20210903193143889](C:\Users\lee\AppData\Roaming\Typora\typora-user-images\image-20210903193143889.png)

### Shared buffer mode

<img src="C:\Users\lee\AppData\Roaming\Typora\typora-user-images\image-20210905155653722.png" alt="image-20210905155653722"  />

## Writting  writeBooster Buffer

+  Writes to the WriteBooster Buffer may decrease the lifetime and the availability of the WriteBooster Buffer.
+  This available buffer size is decreased by the WriteBooster operation and increased by the WriteBooster Buffer flush operation. 
+ The WriteBooster Buffer lifetime is indicated by the bWriteBoosterBufferLifeTimeEst attribute. 

## Flushing WriteBooster Buffer

+ When the entire buffer for WriteBooster is consumed, data will be written in normal storage instead of in the WriteBooster Buffer.
+ The device shall execute the WriteBooster flush operation only when the command queue is empty. If the device receives a command while flushing the WriteBooster Buffer, the device may suspend the flush operation to expedite the processing of that command. 

## Preserve WriteBooster Buffer

+ There are two user space configuration options: “user space reduction” and “preserve user space”. 
+ With the “user space reduction”, the WriteBooster Buffer reduces the total configurable user space. 
+ While with the “preserve user space”, the total space is not reduced. However, the physical storage allocation may be smaller than total capacity of all logical units, since part of the physical storage is used for the WriteBooster Buffer. 
+ When the physical storage allocated for the logical units is fully used, the device will start using the physical storage allocated for the WriteBooster Buffer. 

+ **The amount of performance degradation due to returning the storage used for WriteBooster Buffer to user storage space is device specific. This is because the condition for migration between user space and the WriteBooster Buffer depends on device capacity and vendor’s migration policy, which includes the granularity and frequency of migration, what percentage of free user space remains from which the migration starts, etc. For more details, see device data sheet.**

