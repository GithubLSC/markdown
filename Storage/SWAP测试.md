### 查看swap使用情况

```shell
#列举swap空间使用情况
for i in $( cd /proc;ls |grep "^[0-9]"|awk ' $0 >100') ;do awk '/Swap:/{a=a+$2}END{print '"$i"',a/1024"M"}' /proc/$i/smaps 2>/dev/null ; done | sort -k2nr | head -10

## 根据进程pid 返回详细信息
ps -ef|grep  18906

## 快速统计进程消耗物理内存情况
ps aux | awk '{print $6/1024 " MB\t\t" $11}' | sort -n

## 查看进程虚拟内存映射细节
pmap pid -X
```

#### TPC-C MySQL

```sql
mysqladmin create tpcc -uroot -p12345

mysql tpcc -uroot -p123 < create_table.sql

mysql tpcc -uroot -p123 < add_fkey_idx.sql

./tpcc_load -h127.0.0.1 -d tpcc -uroot -p123 -w1

./tpcc_start -h127.0.0.1  -P3306 -d tpcc -uroot -p123 -w1 -r10 -c32 -l120

```

#### ZRAM

```she
modprobe zram
echo $((1024*1024*1024)) > /sys/block/zram0/disksize
mkswap /dev/zram0
swapon /dev/zram0
```

#### ZSWAP



#### TRACE

20211220

To do lists:

Testing

- [ ] filebench performance on 670p under a variety of device utilizations.

Coding

- [ ] bcc
- [ ] python subprocess,fallocate
- [ ] fragpicker

App launching

- [ ] the worst case of App launching
- [ ] the margin of migration of critical data



