

# 混合式存储的IO行为特征测试

### 1. 文件系统层测试

#### 【测试目的】

**1. 比较三种文件系统下混合式存储的基本读写性能**

**2. 比较三种文件系统下混合式存储的读写混合负载性能表现**

**3. 比较三种文件系统下混合式存储与非混合式存储的性能差异**

**4. 比较容量变化对混合式存储的性能影响**

#### 【测试环境】

|   系统   | Ubuntu 16.04.7 LTS             |
| :------: | :----------------------------: |
| **内核** | 4.15.0-142-generic             |
| <br>**文件系统** | 1. Ext4<br>2. F2FS<br>3. Btrfs |
| <br>**测试工具** | 1. IOzone 3.429<br>2. Filebench 1.5-alpha3<br>3. fio 2.2.10 |

#### 【测试脚本及说明】

1. *工具介绍*

    > fio 是存储设备IO的压力测试工具，用来生成块设备层各种类型的读写负载。
    >
    > **Iozone** 是一个文件系统基准测试工具。用于生成各种基本文件操作并测量性能。
    >
    > **Filebench**是一个文件系统和存储基准测试工具，用于模拟产生应用程序产生的文件系统层负载。
    >
    > blktrace 是一种块层 IO 跟踪工具，用于统计每一条请求的延迟信息。
    >
    > **debugfs 是一个交互式文件系统调试器，用于确定每一个sector对应的具体文件。
    
    ```shell
    apt install f2fs-tools btrfs-tools
   #文件系统格式化
   mkfs.ext4 /dev/nvme0n1
   mkfs.f2fs /dev/nvme1n1
   mkfs.btrfs /dev/nvme2n1
   cd / && mkdir ext4 f2fs btrfs
   #挂载文件系统
   mount /dev/nvme0n1 /ext4
   mount /dev/nvme1n1 /f2fs
   mount /dev/nvme2n1 /btrfs
   ```
   
2. *Fio安装配置使用*

    ```shell
    apt install fio
    #填充数据可采用fio,下例表示对nvme0n1设备填充16g的数据
    eg： fio -filename=/dev/nvme0n1 -ioengine=libaio -direct=1 -name=fill -bs=1m -iodepth=256 -rw=write -size=16g 
    ```

3. *IOzone安装配置使用*

   ```shell
   apt install iozone3
   #测试8g文件的顺序读写、随机读写性能
   eg：iozone -s 8G -f /tmp/iozone -i 0 -i 1
   ```

4. *Filebench安装配置使用*

    ```shell
    #安装
    apt install gcc flex bison
    wget https://sourceforge.net/projects/filebench/files/1.5-alpha3/filebench-1.5-alpha3.tar.gz
    sudo tar -zxf filebench-1.5-alpha3.tar.gz -C /usr/local
    cd /usr/local/filebench-1.5-alpha3
    ./configure
    make && make install
    #测试
    cd /usr/local/share/filebench/workloads
    eg：filebench -f fileserver.f
    #测试负载见附件
    #实际测试需要修改 fileserver.f 等测试文件的测试目录
    eg：set $dir=/ext4
    ```
    > PS：测试负载基本介绍见下表
    
    | Workloads  | Nfiles | Filesize | threads | R/W/A/D | Fsync |
    | :--------: | :----: | :------: | :-----: | :-----: | :---: |
    | Fileserver | 60000  |   128K   |   50    | 1/1/1/1 |   N   |
    |  Varmail   |  1000  |   16K    |   16    | 2/0/2/1 |   Y   |
    |  Webproxy  | 10000  |   16K    |   100   | 5/0/1/1 |   N   |

### 2. 通用块层测试

#### 【测试目的】

**1. 探究混合式存储的写性能问题**
**2. 探究混合式存储的读性能问题**
**3. 探究混合式存储的读写冲突问题**

#### 【测试设备】

|   Device    |   Crucial P1    |   Intel 665p    |   intel 670p    |
| :---------: | :-------------: | :-------------: | :-------------: |
| Flash Type  | Micron 64L QLC  |  Intel 96L QLC  | Intel 144L QLC  |
| Controller  |    SM2263ENG    |    SM2263ENG    |     SM2265G     |
| Form-Factor | PCIe 3.0x4,NVMe | PCIe 3.0x4,NVMe | PCIe 3.0x4,NVMe |
|    DRAM     |    1GB DDR3     |   256MB DDR3    |   256MB DDR3    |
|  Capacity   |       1TB       |       1TB       |       1TB       |

#### 【测试脚本及说明】

1. *大量非更新数据写性能*

   ```shell
   #探究不同容量的设备持续写性能
   #每次测试前需要使用fio 填充数据到一定的容量，然后闲置5 mins，等待内部数据迁移完成
   fio QLCWrite.conf 
   #根据输出文件write_bw.log，观察写带宽变化情况
   ```

2. *大量更新数据写入性能*

      ```shell
      #探究SLC内部GC对写性能的影响
      #为了避免SLC空间过大的问题，建议执行本项测试前填充数据，直至器件内部只有静态SLC (空间占用率80%以上)
      fio SLCGC.conf 
      #根据输出文件write_clat.log，观察写请求延迟变化情况
      ```

3. *混合式设备的读性能*
   
      ```shell
      #混合式存储有SLC和QLC两个区域，数据读取延迟取决于数据的存放区域
      #为了避免SLC空间过大的问题，建议执行本项测试前填充数据，直至器件内部只有静态SLC (空间占用率80%以上)
      . Read.sh
      #比较SLCRead 和QLCRead 两个输出文件的延迟分布情况
      ```
      
4. *读写冲突*

      ```shell
      #探究混合式设备的读写冲突问题
      fio RW.conf 
      #根据输出结果Clat，分析读延迟的分布情况
      #为了测出内部数据迁移对读性能的影响，可以在大量写入后立即开展读写冲突测试
      ```

### 3. 应用启动运行IO行为特征测试


#### 【测试目的】
**1. 探究启动阶段IO行为特征**
**2. 探究浏览器运行阶段读性能尾端延迟特征**

#### 【测试环境】

| **系统** |              Ubuntu  16.04.7 LTS               |
| :------: | :--------------------------------------------: |
| **内核** |               4.15.0-142-generic               |
| **应用** | Firefox  Browser 88.0  <br>LibreOffice-5.1.6.2 |
| **工具** |     Blktrace  2.0.0 <br> Debugfs  1.42.13      |
| **存储** |          Intel  670p 512G (pSLC+QLC)           |

#### 【测试脚本及说明】

1. *LibreOffice Writer启动IO行为特征*

> 1. 利用blktrace对系统盘进行记录,同时启动应用
>
> ```shell
> blktrace -d /dev/nvme1n3p2
> ```
>
> 2. 利用blkparse对trace文件进行分析处理
>
> ```shell
> blkparse -i nvme1n3p2 -d nvme1n3p2.blktrace.bin
> blkparse nvme1n3p2.blktrace.bin > output.txt
> btt -i nvme1n3p2.blktrace.bin -Q queue
> ```
>
> 3. 根据系统进程 compiz 起始筛选出启动过程的请求
> 4. 根据LBA利用debugfs确定每条请求对应的文件信息
>
> ```shell
> debugfs /dev/nvme1n3p2
> icheck LBA(134314) #根据LBA查找对应文件的inode
> ncheck inode(122)  #根据inode查找对应的文件目录
> ```
>
> > 259,6    1        6     1.920453479  1629  D  RS 1037569744 + 32 [compiz] /usr/share/applications/libreoffice-writer.desktop
> > 259,6    7        7     1.920774504  1629  D  RS 1037569776 + 16 [compiz] /usr/share/applications/libreoffice-writer.desktop
> > 259,6    2        9     1.923599944  1323  D RSM 609290504 + 8 [bamfdaemon] /home/bzp
> > 259,6    2       15     1.930263898  2405  D  RS 983292792 + 16 [compiz] /usr/lib/libreoffice/program/bootstraprc
> > 259,6    6       19     1.930494239  2405  D  RS 982523136 + 8 [compiz] 
> >
> > ···
>
> 5. 根据blkparse的输出queue**.dat 文件统计所有请求的大小和当时的队列深度
>
> > number  pro    startstamp sector queuue
> > 1	[compiz]	1.920453	32	1
> > 2	[compiz]	1.920775	16	1
> > 3	[bamfdaemon]	1.9236	8	1
> > 4	[compiz]	1.930264	16	1
> > 5	[compiz]	1.930494	8	1
> > 6	[compiz]	1.930508	8	2
> > 7	[compiz]	1.930515	8	3
> >
> > ···
>
> 注：output.txt为启动trace，blktracename为请求与文件名的对应关系， QS为统计信息

1. *Firefox browser运行读性能尾端特征* 

> 1. 限制主机内存使用
>
> ```sh
> vi /etc/default/gurb
> #grub文件如下
> GRUB_CMDLINE_LINUX_DEFAULT="quiet splash mem=2G "#修改对应行,限制主机内存为2G
> update-grub
> reboot #重启后内存限制生效
> ```
>
> 2. 启动多个应用,利用==`free -h`==命令查看内存使用情况,直至发生SWAP
> 3. 利用blktrace对系统盘进行记录,同时运行firefox browser 开展浏览搜索活动
>
> ```shell
> blktrace -d /dev/nvme0n1
> ```
>
> 4. 利用blkparse对trace文件进行分析处理
>
> 5. 根据进程名 [firefox] 筛选出启动过程的请求
> 6. 统计队列深度信息以及每条请求的延迟信息
> 7. 分析128K读请求的延迟并按延迟排序,结果见附件
>
> > starttime            LBA       sector  queue   latency
> > 46.640638	28195568	256		1	0.000221
> > 83.719834	22611968	256		1	0.000223
> > 95.716537	27655840	256		2	0.000223
> > 45.544130	28267144	256		2	0.000233
> > 45.430245	28247880	256		1	0.000239
> > 52.233318	27626664	256		1	0.000243
> > 61.542621	28330792	256		1	0.000249
> > 312.958144	27979136	256		2	0.000251
> > 45.440017	28116656	256		1	0.000252
>
> 注：附件T是启动过程128k读请求的延迟分布情况

