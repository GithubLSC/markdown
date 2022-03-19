# Script Language

## Shell

### declare

+ [declare](https://www.runoob.com/linux/linux-comm-declare.html) 关键字

> + **declare [+/-] [options] name **
>
> > -a 变量声明为数组
> >
> > -i 变量声明为整形
> >
> > -x 变量声明为环境变量
> >
> > -r 变量声明为只读变量
> >
> > -p 查看变量被声明的类型
### eval
+ [eval](https://blog.csdn.net/her__0_0/article/details/65938894?utm_medium=distribute.pc_relevant_t0.none-task-blog-2%7Edefault%7EBlogCommendFromMachineLearnPai2%7Edefault-1.control&depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-2%7Edefault%7EBlogCommendFromMachineLearnPai2%7Edefault-1.control) 关键字 shell 在执行命令行前扫描它两次

```sh
eval command-line 
pipe="|"
eval ls $pipe wc -l
```
### xargs
+ [xargs](http://www.ruanyifeng.com/blog/2019/08/xargs-tutorial.html) 将标准输入转化为命令行参数

`echo "hello world"|xargs echo`
### seq

+ [seq](https://www.cnblogs.com/faberbeta/p/linux-shell007.html) 序列输出

> 用法：seq [选项]... 尾数
> 　或：seq [选项]... 首数 尾数
> 　或：seq [选项]... 首数 增量 尾数
> 从给定首数到尾数依次进行打印，每两个数字之间的增量等于给定增量。
>
> 必选参数对长短选项同时适用。
>   -f, --format=格式        使用 printf 样式的浮点格式
>   -s, --separator=字符串   使用指定<字符串>分隔数字（默认使用：\n）
>   -w, --equal-width        在列前添加 0 使得宽度相同
>       --help            显示此帮助信息并退出
>       --version         显示版本信息并退出
>
> ```sh
> root@bzp-OptiPlex-7050:/home/bzp# seq 2 5
> 2
> 3
> 4
> 5
> ```

+ [jobs](https://linux.cn/article-6898-1.html) 显示正在进行的作业
+ [&&和||](https://blog.csdn.net/htlin1990/article/details/80887346) command 逻辑运算符

```sh
[root@ol01 htlin]# [[ -e "/tmp/htlin/htlin.txt" ]] && echo "file exits"
file exits

[root@ol01 htlin]# [[ ! -e "/tmp/htlin/htlin.txt" ]] && echo "file not exits"
file not exits

```

***

```shell
[root@ol01 htlin]# [[ -e "/tmp/htlin/htlin.txt" ]] ||  echo "file not exits"
file not exits
```
### serial && parellel
+ [shell并行](https://blog.csdn.net/yangshangwei/article/details/87691890)  串行执行和并行执行
  + shell命令串行执行 假定业务多个业务逻辑没有先后关系，每个脚本执行时间很长，推荐并行执行

> 串行
>
> ```sh
> [root@artisan test]# cat call_serial.sh 
> #!/bin/bash
> #当前目录下执行如下脚本  相对路径
> ./1.sh 
> ./2.sh 
> echo "继续执行剩下的逻辑..."
> ```
>
> 并行(==&==使得脚本在后台运行，无需等待当前进程结束，最后务必使用==wait==关键字，用来确保所有子进程都执行完成)
>
> ```sh
> [root@artisan test]# cat call_parallel.sh 
> #!/bin/bash
> #当前目录下执行如下脚本  相对路径
> ./1.sh &
> ./2.sh &
> wait
> echo "继续执行剩下的逻辑..."
> ```
## Python
### argparse

+ [argparse ](https://zhuanlan.zhihu.com/p/56922793)python 的命令行解析标准模块，直接在命令行中可以向程序传入参数并让程序运行

```python
import argparse

parser = argparse.ArgumentParser(description='命令行中传入一个数字')
#type是要传入的参数的数据类型  help是该参数的提示信息
parser.add_argument('integers', type=str, help='传入的数字')

args = parser.parse_args()

#获得传入的参数
print(args)
```

```python
import argparse

parser = argparse.ArgumentParser(description='姓名')
parser.add_argument('--family', type=str,help='姓')
parser.add_argument('--name', type=str,help='名')
args = parser.parse_args()

#打印姓名
print(args.family+args.name)

python demo.py --family=张 --name=三
```
