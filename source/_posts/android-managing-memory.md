title: Android-Managing-Memory
categories:
	- Android
tags:
	- Optimization
date: 2015-11-29 14:30:58
---
Android App上内存管理及优化方式。

## 一、 How Android Manages Memory
Android does not offer swap space for memory, but it does use paging and memory-mapping (mmapping) to manage memory. 所以，唯一的释放内存的方式就是释放对象的引用。

***That is with one exception***: any files mmapped in without modification, such as code, can be paged out of RAM if the system wants to use that memory elsewhere.

### 1. Sharing Memory

Share RAM pages across processes in the following ways:

1. Zygote进程加载的framework代码和资源，是可以在所有app zygote进程中共享的。Each app process is forked from an existing process called Zygote. The Zygote process starts when the system boots and loads common framework code and resources (such as activity themes). To start a new app process, the system forks the Zygote process then loads and runs the app's code in the new process. This ***allows most of the RAM pages allocated for framework code and resources to be shared across all app processes***.
2. ***Most static data is mmapped into a process***. This not only allows that same data to be shared between processes but also allows it to be paged out when needed. Example static data include: Dalvik code (by placing it in a pre-linked ***.odex file*** for direct mmapping), ***app resources*** (by designing the resource table to be a structure that can be mmapped and by aligning the zip entries of the APK), and traditional project elements like native code in ***.so files***.
3. 显式分配的共享内存。In many places, Android shares the same dynamic RAM across processes using explicitly allocated shared memory regions (either with ashmem or gralloc). For example, window surfaces use shared memory between the app and screen compositor, and cursor buffers use shared memory between the content provider and client.

### 2. Allocating and Reclaiming App Memory

1. Dalvik heap是一块虚拟的内存区间，也就是逻辑堆，大小可以增长，但是系统给每个app设定了一个上限；
2. The logical size of the heap is not the same as the amount of physical memory used by the heap. 系统会计算PSS的大小，作为物理内存大小。
3. The Dalvik heap does not compact the logical size of the heap. 当heap的内存尾端不够的时候，dalvik会扫描整个heap 找到不使用的pages回收。

### 3. Restricting App Memory

Android sets a hard limit on the heap size for each app. If your app has reached the heap capacity and tries to allocate more memory, it will receive an OutOfMemoryError.

如果想要app最多能使用多少内存，使用***getMemoryClass()***，返回interger百万字节数。

### 4. Switching Apps

low memory的时候，会kill LRU least-recently used process.但是也会考虑那些占用内存很大的process。

## 二、 How Your App Should Manage Memory

### 1. Activity、Service组件

##### 1. Use services sparingly

1. 一个后台运行的service会以服务进程优先级占用ram，从而减少系统在lrucache缓存的process。如非必要，不要运行；完成工作后，一定确保stop。否则会导致leak service。Leaving a service running when it’s not needed is one of the worst memory-management mistakes。
2. 解决办法：使用***IntentService***，它会在完成工作后销毁自己，即在handleMessage中调用stopSelf()；IntentService原理HandlerThread+Handler实现异步。

##### 2. Release memory when your user interface becomes hidden

当用户exit你的app，即所有的UI组件都hidden的时候，系统回调onTrimMemory();释放有关ui展示的资源:比如图片，文件，动态生成的view等。

可以实现onTrimMemory()的组件:

* Application.onTrimMemory()
* Activity.onTrimMemory()
* Fragement.OnTrimMemory()
* Service.onTrimMemory()
* ContentProvider.OnTrimMemory()


***onTrimMemory()*** : api 14(android 4.0 api 15)之后加入的，callback with ***TRIM_MEMORY_UI_HIDDEN*** only when all the UI components of your app process become hidden from the user.    
更多level标签：http://developer.android.com/training/articles/memory.html#YourApp    
***onStop()*** : 在App内跳转也会执行，to release activity resources such as a network connection or to unregister broadcast receivers, you usually should not release your UI resources until you receive onTrimMemory(TRIM_MEMORY_UI_HIDDEN).

### 2. BitMap

##### 1. Check how much memory you should use

***getMemoryClass()*** to get an estimate of your app's available heap in megabytes. If your app tries to allocate more memory than is available here, it will receive an OutOfMemoryError.

***getLargeMemoryClass()*** In very special situations, call this to get an estimate of the large heap size, setting the largeHeap attribute to "true" in the manifest <application> tag. 最好永远不要用。

##### 2. Avoid wasting memory with bitmaps

***Keep it in RAM only at the resolution you need*** for the current device's screen, scaling it down if the original bitmap is a higher resolution.
更多参考： Android-Displaying-Bitmap-Efficiently

### 3. 变量类型使用

##### 1. Use optimized data containers

***Take advantage of optimized containers*** in the Android framework, ***such as***

* SparseArray
* SparseBooleanArray
* SparseIntArray
* SparseLongArray
* LongSparseArray

***instead of HashMap*** which can be quite memory inefficient because it needs a separate entry object for every mapping. 

为何SparseArray要比hashMap要好，both because it avoids
 * auto-boxing keys and its data structure doesn't rely on an extra entry object for each mapping.因为他不需要autoboxing(即将原始类型封装为对象类型，比如把int类型封装成Integer类型）。***当数据量不多(几百)的时候，用SparseArray。但是数据量多hashmap效率较高。***原因是SparseArray存储用了两个数组int[] keys和Object[] values;查找的时候通过二分法查找，添加删除涉及到元素移动。

##### 2. Be aware of memory overhead

1. Enums often require more than twice as much memory as static constants. You should strictly avoid using enums on Android.
2. Every class in Java (including anonymous inner classes) uses about 500 bytes of code.
3. Every class instance has 12-16 bytes of RAM overhead.
4. Putting a single entry into a HashMap requires the allocation of an additional entry object that takes 32 bytes (see the previous section about optimized data containers).

##### 3. 数组

1. int[]效率高于Integer[];
2. 两个并行的数组比一个（int,int）的Object数组好太多；也就是两个ObjA[]和ObjB[]比一个(ObjA,ObjB)[]效率好很多；

##### 4. 注意使用Float

float比int运行速度慢2倍；float和double运行速度差不多，double空间比float大2倍；在desktop开发上，不考虑空间的时候使用double。

### 4. 编码Tips
##### 1. 避免创建不必要的对象
1. String str = new String("Hello"); 改为 String str = "Hello";
2. 方法A返回的string一定会append到StringBuffer上，就直接append，不要再创建临时变量；
3. 循环外定义变量；

##### 2. Static Final for constants

static int VAL = 10; 会通过clinit的初始化方法，保存10到变量VAL中；访问变量的时候通过引用查找变量；

static final VAL = 10; 不会调用clinit方法，使用时会直接使用10的值。

##### 3. 避免internal的get/set

Internal的get/set在C＋＋，Java，C＃中都是良好的面向对象的编程习惯。但是在Android上，要直接访问变量。Without a JIT, direct field access is about 3x faster than invoking a trivial getter. With the JIT (where direct field access is as cheap as accessing a local), direct field access is about 7x faster than invoking a trivial getter.

Note that if you're using ProGuard, you can have the best of both worlds because ProGuard can inline accessors for you.

##### 4. Use Enhanced For Loop Syntax

```
int sum = 0;
    for (Foo a : mArray) { // jdk 1.5之后
        sum += a.mSplat;
    }

```
##### 5. 使用Lib效率更高

String.indexof()
System.arraycopy()

### 5. Be careful with code abstractions

抽象可以更加敏捷和便于维护。然而抽象会有they require a fair amount more code that needs to be executed, requiring more time and more RAM for that code to be mapped into memory. 所以如果抽象并不是具有明显的意义，不要使用。

### 6. 引用第三方

##### 1. Use nano protobufs for erialized data
使用[PB nano版本](https://android.googlesource.com/platform/external/protobuf/+/master/java/README.txt)，普通的版本会产生繁琐的code，导致内存使用增加，包大小增加，执行缓慢，还有可能出现64k方法limit的问题。


##### 2. Avoid dependency injection frameworks

Using ***a dependency injection framework*** ***such as Guice or RoboGuice*** may be attractive because they can simplify the code you write and provide an adaptive environment that's useful for testing and other configuration changes. However, ***these frameworks tend to perform a lot of process initialization by scanning your code for annotations***, which can require significant amounts of your code to be mapped into RAM even though you don't need it. 

##### 3. Be careful about using external libraries
注意lib的大小，以及内存占用情况。如果针对app的，注意一下兼容情况，比如用的nano pb。

### 7. 编译打包过程

##### 1. Use ProGuard to strip out any unneeded code

The ProGuard tool shrinks, optimizes, and obfuscates your code by removing unused code and renaming classes, fields, and methods with semantically obscure names. Using ProGuard can make your code more compact, requiring fewer RAM pages to be mapped.

##### 2. Use zipalign on your final APK

签名后必须zipalign。Failing to do so can cause your app to require significantly more RAM, because things like resources can no longer be mmapped from the APK.

### 8. Use multiple processes

***Most apps should not run multiple processes***, as it can easily increase—rather than decrease—your RAM footprint if done incorrectly.

An example of when multiple processes may be appropriate is when building a music player that plays music from a service for long period of time. If the entire app runs in one process, then many of the allocations performed for its activity UI must be kept around as long as it is playing music.

***new process***
```
<service android:name=".PlaybackService"
         android:process=":newprocessname" />
```

***Concern***
1. 两个process，一个处理ui，另一个与ui无关。
2. 注意代码规范，比如如果用了enum，此时2个进程占用的内存就会双份。
3. 注意两个进程间的依赖，不能有任何的content provider或者service的依赖：For example, if your app has a content provider that you have running in the default process which also hosts your UI, then code in a background process that uses that content provider will also require that your UI process remain in RAM. If your goal is to have a background process that can run independently of a heavy-weight UI process, it can't have dependencies on content providers or services that execute in the UI process.