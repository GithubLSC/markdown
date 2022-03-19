# JAVA基础

## 路线图
> https://www.bilibili.com/read/cv5702420

1. Java SE
2. 数据库
3. 前端
4. Java Web
5. SSM框架
6. Linux



1. SpringBoot
2. SpringCloud
3. Hadoop

## Java介绍

### 编程语言发展史

> 1972 C
>
> 1982 C++
>
> 1995 Java 

### Java 发展

+ javaSE
+ ~~javaME~~
+ javaEE

构建工具：Ant Maven

应用服务器 Tomcat Jetty

Web开发 Spring myBatis

开发工具 Eclipse

大数据 Hadoop 2006

手机端 Android 2008

### Java 特性

+ JVM Write Once，Run Anywhere！
+ 反射  

### JDK JRE JVM

### 运行机制

+ 编译型
+ 解释型

## Java 基础

### 注释

+ 单行注释
+ 多行注释
+ 文档注释

### 标识符

### 数据类型

+ 强类型语言：*要求变量的使用要严格符合规定，所有变量必须先定义，后使用*

+ 数据类型

  + 基本类型 primitive type

    + byte
    + short
    + int
      + 0b
      + 0
      +  
      + 0x
    + long
    + float

    > 最好完全使用浮点数进行比较
    >
    > BigDecimal（数学工具类）

    + double
    + char       
    + boolean

    > string 不是关键字,属于类

+ 引用类型 reference type
  + 类
  + 接口
  + 数组
+ 类型转换
  + 强制类型转化
  + 自动类型转换

### 变量

+ 实例变量
+ Static 类变量
+ 局部变量

+ 常量 
  + final
  + 一般用大写字母
+ 命名规则
  + 局部变量、类变量、方法名 *首字母小写，驼峰命名*
  + 常量 *大写字母，下划线*
  + 类名 *首字母大写，驼峰命名*

### 运算符

+ 算术运算符

  > 元素有long 则结果为long
  >
  > 元素无long 则结果为int

+ 赋值运算符

+ 关系运算符 instanceof

+ 逻辑运算符

+ 位运算符

+ 条件运算符

+ 扩展赋值运算符

### 包机制

为了更好地组织类，Java提供包机制，用于区别类名的命名空间

一般利用公司域名倒置作为包名

import package1[.package2] (classname.*)

### Java Doc

javadoc命令用来生成自己的API文档

> @author
>
> @version
>
> @since
>
> @param
>
> @return
>
> @throws 

```sh
javadoc hello.java
```

## 流程控制

### 用户输入

 + *Scanner*

    + next() hasNext()

      > 不含空格
      >
      > 一定读到有效字符后结束

    + nextLine() hasNextLine()

      >  可以含空格
      >
      > 可以获得空白

### 顺序结构

### 选择结构

> IDEA 反编译

### 循环结构

> 100.fori =for(int i=0;i<100;i++){}
>
> Alt+Enter 自动补全import
>
> psvm
>
> sout

### 方法

+ Java 值传递

+ 可变参数

  ```java
  public void test(int... i){
          System.out.println(i[0]);
      }
  ```

+ 递归

  + 递归头
  + 递归体


## 面向对象编程

> 以类的方式组织代码，以对象方式组织封装数据

### 三大特性

+ 封装
+ 继承
+ 多态

### 方法

+ 构造器

  + 每个类至少存在默认的构造方法
  + 与类名相同，没有返回值 
  + alt+insert生成默认构造器

+ 堆栈方法区

+ 封装

  + ==高内聚，低耦合==
  + alt+insert 自动生成get set方法

+ 继承

  继承的本质是对某一批类的抽象

  extends

  Java种只有单继承，没有多继承

  所有的类都直接或间接继承Object

  super和this 

  父类的构造器必须在子类构造器的第一行

+ 方法的重写

  + public>protected > default><private
  + 需要继承关系 子类重写父类
  + 非private才可以重写
  + 非静态重写
  + 重写异常抛出范围可以缩小，不能扩大
  + alt+insert 可自动生成重写函数体

+ 多态 

  + 父类的引用指向子类对象
  + 方法的多态 同一个方法可以根据发送对象的不同而采用不同对象行为方式
  +  ClassCastException
  + 存在条件
    + 继承关系
    + 子类重写父类方法
    + 父类引用指向子类的对象

+ Instanceof  父子关系 

+ 代码块

  + 静态代码块

  + 匿名代码块

  + 构造方法

    ```java
    import static java.lang.Math.random;//静态导入包
    ```

+ 抽象类

  + abstract 关键字
  + 抽象类的所有方法必须由子类实现
  + 不能new 
  + 抽象类中可以包含普通方法
  + 抽象方法必须在抽象类中
  + **抽象类存在构造器吗** 存在

+ 接口                    

  + 接口可以多继承
  + 只有规范 方法定义
  + 接口中的方法默认 public abstract 

+ 内部类

  + 成员内部类

    获得外部类的私有属性

  + 静态内部类

  + 局部内部类

    一个java文件可以有多个class但智能有一个public class

  + 匿名内部类

  + Lambda 表达式

+ 异常

  + 体系架构

  + 异常和错误的区别

    Error往往是灾难性错误，是程序无法控制和处理的，当出现时，JVM一般会选择终止线程；Exception通常可以被程序处理，并在程序中尽可能的去处理这些异常。

    java把异常当作对象处理，并定义一个基类java.lang.Throwable作为所有异常的超类

    <img src="C:\Users\lee\AppData\Roaming\Typora\typora-user-images\image-20210710200650819.png" alt="image-20210710200650819" style="zoom: 67%;" />

    + 检查性异常
    + 运行时异常
    + 错误

  + 异常处理机制

    + 抛出异常

    + 捕获异常

    + 关键字

      + try
      + catch
      + finally
      + throw
      + throws

      ```java
      try {
          System.out.println(a/b);
      }catch(ArithmeticException e){
          System.out.println("程序出现异常！");
      }finally{
          System.out.println("finally");
      }
      ```

    + ctrl+ALt+t 自动包裹代码

## 数组

### 定义

+ 静态初始化

+ 动态初始化

  ```java
  int[] array ={1,2};
  int[] array=new int[3];
  array.for 
  ```

+ Arrays类

  ```java
  Arrays.toString(a)
  ```

+ 稀疏数组

  ---

  > https://www.bilibili.com/video/BV1CJ411m7gg?p=85

## 集合

### 特点

+ 集合可以存放不同的类型，不限数量的数据
  + Set
  + List
  + Map

### Hashset

**Collection接口$\rightarrow$ set 接口$\rightarrow$ HashSet类**

当向HashSet集合中存放一个元素时，hashset会调用该对象的HashCode()方法来获得该对象的Hashcode值，然后根据hashcode值决定该对象在hashset中的位置。两个元素的equals()不同时，hashset可以添加成功。

+ 不能保证元素的排列顺序
+ 不可重复
+ 不是线程安全
+ 集合元素可以是null

### TreeSet

**Collection接口$\rightarrow$ set 接口$\rightarrow$ SortedSet接口 $\rightarrow$ NavigableSet 接口$\rightarrow$ TreeSet类****

TreeSet是SortedSet接口的实现类，可以确保集合元素处于排序状态。

+ 自然排序
+ 定制排序
  + 重写Comparator接口中的compare方法

### List与ArrayList与LinkedList

**Collection接口$\rightarrow$ List 接口$\rightarrow$ ArrayList类**

元素有序，可重复

### HashMap

保存具有映射关系的数据

HashMap和HashTable是Map接口的两个典型的类实现

+ 区别
  + Hashtable线程安全 HashMap线程不安全
  + Hashtable不允许使用Null作为key或者value

### TreeMap

跟TreeSet功能类似

自然排序是字典排序

## Collections

操作集合工具类

+ 排序
  + reverse(List)
  + shuffle(List)
  + sort(List)
  + sort(List,Comparator)
  + swap(List,int,int)
  + max
  + min



## IO流

### 流的分类

按数据单位不同

+ 字节流（8bit）
+ 字符流（16bit）

按数据流向

+ 输入流
+ 输出流

按流的角色

+ 结点流
+ 处理流

<img src="C:\Users\lee\AppData\Roaming\Typora\typora-user-images\image-20210712200955140.png" alt="image-20210712200955140" style="zoom:50%;" />

### 文件流

> 基于磁盘

#### 文件字节流

+ FileInputStream
+ FileOutputStream

#### 文件字符流

==只适合操作内容是字符的文件。==

+ FileReader
+ FileWriter

### 缓冲流

> 基于内存

+ BufferedInputStream
+ BufferedOutputStream
+ BufferedReader
+ BufferedWriter

### 转换流

> 字节流 字符流 需要指定编码格式

+ InputStreamReader
+ OutputStreamWriter

### 标准输入输出流

+ System.in
+ System.out

### 数据输入输出流

+ DataInputStream

  ```java
  DataInputStream in=new DataInputStream(new FileInputStream("src/new.txt"));
  in.readInt();
  ```

+ DataOutputStream

    ```java
    DataOutputStream out=new DataOutputStream(new FileInputStream("src/new.txt"));
    out.WriteInt(1);	
    ```

### 对象流

序列化不能处理static和transient修饰的成员变量

```java
public class Person Implements Serializable{
    public String name;
    public int age; 
}
//序列化，反序列化包必须一致
```

+ ObjectInputSteam

  序列化 Serialize

+ ObjectOutputSteam

  反序列化 Deserialize

### 随机存取流

RandomAccessFile

## File 类

file能新建删除重命名文件目录，但不能访问文件内容本身。

```java
//递归遍历当前目录下所有文件
public static void traverse(File file){
        if(file.isDirectory()){
            System.out.println("filedir:"+file.getName());
            File [] files=file.listFiles();
            if(files!=null&&files.length>0){
                for(File i:files){
                    traverse(i);
                }
            }
        }else{
            System.out.println("file:"+file.getName());
        }
    }
```

## 设计模式

大量实践中总结和理论化之后优选的代码结构、编程风格、以及解决问题的思考方式。

### Singleton 单例设计模式

+ 饿汉式

  构造方法私有化    

+ 懒汉式

区别在于什么时候 new 对象

### Template Method 模板方法设计模式

### Factory Method 工厂设计模式

## 泛型Generic

泛型只在编译阶段有效。

### 泛型类

### 泛型方法

方法也可以被泛型化，不管此时定义在其中的类是不是泛型化的，在泛型方法中可以定义泛型参数，此时，参数的类型就是传入数据的类型。

### 泛型接口

未传入泛型实参时，与泛型类的定义相同，在声明类时，需将泛型的声明也一起加入类中。

### 通配符

?

#### <? extends Person>

只允许泛型为Person 及其子类的引用调用

#### <? super Person>

只允许泛型为Person 及其父类的引用调用

#### <? extends C>

接口

### 枚举类

### 注解 Annotation

#### 元注解

+ @Target : 用于描述注解的适用范围
+ @Retention：描述注解的生命周期
+ @Document：说明该注解可包含在JavaDoc中
+ @Inherited：说明子类可以继承父类中的该注解



+ Override 限定重写父类方法
+ Deprecated 表示某个程序元素已过时
+ SuppressWarrings 抑制编译器警告

## 反射

### Class类

+ 通过类名.class 创建指定类的Class实例
+ 通过一个类的实例对象的getClass方法，获取对应实例对象的类的Class实例
+ 通过Class的静态方法forName 来获取某个类的Class 实例

### 通过反射获取类的完整结构

   ## java 动态代理

## java 静态代理

+ Proxy 专门完成代理的操作类，是所有动态代理的父类。通过此类为一个或多个接口动态地生成类。

## 线程

### 创建方式

+ Thread Class
+ Runnable 接口
+ Callable 接口

## Lambda 表达式

+ 避免了匿名内部类定义过多
+ 简洁   



