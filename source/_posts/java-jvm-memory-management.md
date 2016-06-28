title: JVM-Memory-Management
categories:
  - Java
tags:
  - Memory
date: 2015-12-09 16:22:28
---
JVM虚拟机的介绍以及Java内存分配管理内容。

## 一、 JVM Basic

每一个程序都对应一个JVM。JVM内部分为两类线程：

1. 守护线程：GC线程
2. 执行线程：main函数线程及其启动的线程

Java程序也可以把他创建的任何线程标记为守护线程。所有的非守护线程退出时，JVM退出。

## 二、 JVM SubSystem

包含四块子系统：

1. 垃圾回收器 Garbage Collection：回收堆内存；
2. 类装载子系统 Classloader Sub-system：定位和导入类文件，并验证被导入类正确性，为类变量分配并初始化内存；
3. 执行引擎 Execution Engine：执行类装载子系统中加载的类,即时编译被执行的字节码成机器代码，放入缓存，以后调用可以重用；
4. 内存区 Java Memory Allocation Area。

JVM的数据类型：

1. 基本类型：
	1. 数值类型
		1. 浮点类型
			1. float
			2. double
		2. 整数类型
			1. byte
			2. short
			3. int
			4. long
			5. char
	2. boolean
	3. returnAddress
		 	
2. 引用类型：
	1. 引用
		1. 类
		2. 接口
		3. 数组类型

## 三、 内存分区

分为5个区域：

1. 程序计数器：JVM支持多个线程同时运行，当每一个新线程被创建时，它都将得到它自己的PC寄存器（程序计数器）。如果线程正在执行的是一个Java方法（非native），那么PC寄存器的值将总是指向下一条将被执行的指令，如果方法是 native的，程序计数器寄存器的值不会被定义。
2. 堆：Java堆是被所有线程共享的一块内存区域。所有的对象实例都是在这里分配内存，但是这个对象的引用却是在栈（Stack）中分配。GC管理堆的对象回收。
3. 栈：又叫堆栈。运行时变量、引用和方法返回。
4. 本地方法栈：存储本地方法的调用状态。
5. 方法区：装载一个class文件的类型信息（包括类信息、常量、静态变量等）放到方法区中，该内存区域被所有线程共享。本地方法区存在一块特殊的内存区域，叫***常量池（Constant Pool)***。


参考文章    
[java内存分配和String类型的深度解析](http://my.oschina.net/xiaohui249/blog/170013)    
[JAVA虚拟机体系结构](http://www.cnblogs.com/java-my-life/archive/2012/08/01/2615221.html)