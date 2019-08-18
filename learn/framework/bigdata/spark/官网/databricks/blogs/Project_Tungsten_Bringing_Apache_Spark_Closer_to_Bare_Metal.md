# Project Tungsten -- spark 执行引擎
*   专注提高cpu 和 内存的利用率，使得利用率接近到现代硬件的上限
####  [原文](https://databricks.com/blog/2015/04/28/project-tungsten-bringing-spark-closer-to-bare-metal.html)

##  Tungsten engine ==> 优化内存和ＣＰＵ的使用，让性能颈瓶为最新计算机的性能，主要针对３点：

####    内存管理和二进制处理，充分利用应用程序语义明确地管理内存和消除JVM的对象模型和垃圾收集的开销

    1.1 4个字节的字符串"abcd",jvm对象中占用４８个字节

java.lang.String object internals:

OFFSET | SIZE  | TYPE DESCRIPTION     |               VALUE
----|-------|-----------------------|-------------------
0   |  4    |    (object header)    |            ...
4   |  4    |   (object header)     |           ...
8   |  4    |    (object header)    |            ...
12  |   4 |char[] String.value       |            []
16  |   4 |   int String.hash        |            0
20  |   4 |   int String.hash32      |            0

Instance size: 24 bytes (reported by Instrumentation API)

    1.2 jvm gc，针对通常的回收机制，对于ｓｐａｒｋ不适用，spark知道数据流在各个阶段的信息，可以更高效的管理

    1.3 spark manager 把大多数ｓｐａｒｋ操作转换为直接对二进制数据的操作而不是java的object

    通过sun.misc.Unsafe提供的Ｃ语言方式的内存访问（allocate,deallocate,pointer)

####    缓存感知计算：利用内存层次结构的算法和数据结构

    2.1使用cpu L1/L2/L3三级缓存

    2.1 CPU很高比例花在从main memory读取数据

    2.3 sort例子－－快速排序（两两比较交换位置）

    　　原来方式　＝＞　pointer  -->　内存对象ｋｅｙ，ｖａｌｕｅ　｜　通过pointer 获取对象　比较ｋｅｙ，排序

    　　内存感知方式　＝＞　key + pointer -->　内存对象value　　｜　排序ｐｏｉｎｔｅｒ－ｋｅｙ就行　不用找对象　

####    代码生成：使用代码生成利用现代编译器和CPU

    3.1 sql 表达式（ａｇｅ＞３５）会被动态生成字节码运行，减少量基本类型的boxing,

    3.2 The Black Magic of (Java) Method Dispatch

           java继承多态可以模块化和代码复用的同时，会产生虚拟调用，硬件无法支持虚拟调用，需要运行时模拟行为

           Abstract类调用比static 调用多null-check
    　　interface＝＝＞check the coder type

    3.3 通过把shffle时候的数据从内存中的二进制格式转换为wire-protocol格式，显著提高速度　　

    　　


性能颈瓶：

    IO和network相比ＣＰＵ和内存的使用已经不是主要问题＝＝＞１网络和硬盘性能提升２程序优化处理

    ｓｐａｒｋ的shuffle系统中serialization and hashing（CPU使用）



whole-stage code generation

    运行时优化字节码，把整个实体的访问转换成单个函数

    消除虚拟函数调用

    保存中间数据到ｃｐｕ寄存器


Volcano Iterator Model-－之前的数据查询方式

    dataflow query execution system

    基于iterator模式，

    拥有简单的接口，任意组合操作

    问题一：每次都调用ｎｅｘｔ

    问题二：next是虚拟调用，开销比普通调用大



Whole-stage Code Generation－－新的方式（运行时生成执行的代码）

    消除虚函数调用

    中间数据从内存转移到ｃｐｕ寄存器中缓存

    最新ＣＰＵ的优化功能：

    ３．１Single Instruction Multiple Data（ＳＩＭＤ）－－是一种采用一个控制器来控制多个处理器，同时对一组数据（又称“数据矢量”）中的每一个分别执行相同的操作从而实现空间上的并行性的技术。

    ３．２循环展开，英文中称（Loop unwinding或loop unrolling），是一种牺牲程序的尺寸来加快程序的执行速度的优化方法。可以由程序员完成，也可由编译器自动优化完成。
    循环展开最常用来降低循环开销，为具有多个功能单元的处理器提供指令级并行。也有利于指令流水线的调度。

    1




vectorization　－－批量处理行
