title: Android-Performance-Tools
categories:
	- Android
tags:
	- Optimization
date: 2016-03-30 15:52:25
---
性能优化工具介绍及使用。    
要显示一个像素，需要4个阶段：（1）CPU完成计算（2）GPU完成渲染（3）内存存储图像像素（4）Battery提供电源。每一个都可能达到瓶颈。要优化，要从多方面开始。

***Note:***

在优化的时候，关闭Instant Run。它会造成性能影响。



## 一. CPU优化

### 1. CPU Monitor

集成在Android Monitor，可以看user mode和kernel mode分别占用的CPU比例。

### 2. Method Trace

##### 1. 使用方式

1. 打开DDMS，在左侧面板上面Start Method Profling，然后再次点击就Stop；
2. 嵌入代码精准定位
	
	```
	android.os.Debug.startMethodTracing(“tracefilename”);
	android.os.Debug.stopMethodTracing();
	
	// adb pull /sdcard/test.trace /tmp 取出trace文件
	```
	
##### 2. 参数简介

1. Incl Cpu Time : 方法本身代码和子方法时间总数；
2. Excl Cpu Time : 方法本身代码执行时间；不包括子方法；
3. Calls + Recur : 调用次数和递归调用次数；
4. Cpu Time/Call : 方法每次调用占用CPU的时间；
5. Real Time : 方法执行完总花费时间，因为有cpu调度消费，比Cpu Time要多；

##### 3. 重点关注

1. 调用次数少，每次花费时间多的；
2. 调用次数多的；


## 二、GPU优化

### 1. GPU Monitor
 
集成在Android Monitor，绿色横线60frames/秒的标准线，红色线30frames/秒的标准线。

### 2. OverDraw

##### 1. Enable in Developer Options
##### 2. Color:

* True Color: no overdraw 代码指定的实际颜色
* Blue: overdraw once  其实有点偏紫色
* Green: overdraw twice 浅绿色
* Pink: overdraw three times
* Red: overdraw four or more times

#### 3. 优化方式
参考Android-Improving-Layout-Performance

### 2. Profiling GPU Rendering

##### 1. Enable in Developer Options : Profile GPU Rendering

##### 2. 原理：
大多数卡顿是因为渲染性能问题。Android每16ms发送VSYNC信号对UI进行渲染，每次渲染即为一帧。如果16ms内没有绘制完，比如在20ms绘制完，则实际上用户在32ms内看到一帧，丢失一帧显示。丢帧太多，就会卡顿。所以要尽量保证所有的操作在16ms内完成。在Android Monitor里的GPU Monitor的红线上可以看到地狱30frames的渲染。
##### 3. 为什么16ms？
人视觉产生连续效果需要达到基本的24fps，电影一般用这个帧数。60fps是人眼能感受到的最快的变化，超过60fps是不必要的。为了达到60fps，即需要16ms绘制一帧。    
每16ms刷新一次的刷新频率，60fps的帧频率，如果刚好匹配，则完美工作。实际上很难刚好保证同步。

1. 帧率超过刷新频率，即在16ms等待VSYNC刷新信号的过程中，帧数据需要等待VSYNC刷新被hold住以保持每次刷新都有数据可以显示；
2. 帧率低于刷新频率。即16ms内没有绘制完一帧，用户在32ms、48ms内看到一帧完整显示。就是卡顿了。

##### 4. Colors

1. 绿线表示16ms； 每一帧超过16ms，就意味着丢失一帧；
2. 帧的颜色从底部到顶部：
	* 绿色：代表创建和更新view的时间；过长表示同时绘制的view过多或ondraw比较费时；
	* 紫色：4.0之上才有，传输资源到渲染线程花费的时间
	* 红色：OpenGL渲染图像花费的时间
	* 黄色：CPU等待GPU绘制结束花费的时间，如果过长，说明GPU工作很多。CPU发送cmd给GPU后，需要等过长时间才能发送新的cmd，会导致黄色很长。
 
## 三、Battery优化
～～～ 没研究。

## 四、内存优化















## 四、 Memory Analysis Tools
通过Memory Monitor观察到频繁的GC，用Heap Viewer看一下对象类型，用Allocation Tracker找到问题代码位置。

### 1. MemoryMonitor

##### 1. 查看内容

1. free和allocated的内存
2. 查看GC garbage collection的频率是否正常，频繁gc导致性能问题。
3. 是否内存溢出导致crash
4. 看看是否有内存泄漏

### 2. Heap Viewer

Android Monitor和DDMS里都有。

＃#### 1. 查看内容
1. 查看分配内存的对象，数量及占内存大小
2. 查看不必要分配的内存对象，检查内存泄漏

### 3. Allocation Tracker

1. 可以帮助找到问题代码的位置。

##### 1. 查看内容


### 4. Investigating Your RAM Usage

#####  1. Interpreting Dalvik Log Messages

1. GC Reason
	* ***GC_CONCURRENT***
    A concurrent GC that frees up memory as your heap begins to fill up.
	* ***GC_FOR_MALLOC***    
    A GC caused because your app attempted to allocate memory when your heap was already full, so the system had to stop your app and reclaim memory.
	* ***GC_HPROF_DUMP_HEAP***    
    A GC that occurs when you request to create an HPROF file to analyze your heap.
	* ***GC_EXPLICIT***    
    An explicit GC, such as when you call gc() (which you should avoid calling and instead trust the GC to run when needed).

##### 2. ART Log Message

http://developer.android.com/tools/debugging/debugging-memory.html#LogMessages

## 五、 Battery Analysis Tools