title: Android-Tools
categories:
  - Android
tags:
  - Optimization
date: 2015-07-09 09:40:09
---
Android tools of sdk and platform.

## 一、 SDK Tools

### 1. Virtual Device Tools

##### 1. Android Virtual Device Manager
##### 2. Android Emulator
##### 3. mksdcard    
Helps you create a disk image that you can use with the emulator, to simulate the presence of an external storage card (such as an SD card).

### 2. Development Tools

##### 1. Hierarchy Viewer

The Hierarchy Viewer allows you to debug and optimize your user interface. It provides ***a visual representation of the layout's View hierarchy (the Layout View)*** and ***a magnified inspector of the display (the Pixel Perfect View)***. 

##### 2. lint
##### 3. SDK Manager
##### 4. sqlite3


### 3. Debugging Tools

##### 1. Android Monitor
##### 2. adb
##### 3. ADB Shell
##### 4. DDMS dalvik debug monitor server
##### 5. Device monitor
##### 6. dmtracedump
##### 7. hprof-conv
##### 8. systrace
##### 9. traceview
##### 10. tracer for OpenGL ES


### 4. Build Tools

##### 1. JOBB
##### 2. aapt

1. ~/sdk/build-tools/xxxx/aapt dump badging xxx.apk 

	查看包的基本信息，如packageid，version，permission等

##### 3. ProGuard

### 5. zipalign

1. 优化方式：保证apk内未压缩的资源文件，比如图片和二进制流文件四子节边界对其；
2. 好处：运行时节省内存。This allows all portions to be accessed directly with mmap() even if they contain binary data with alignment restrictions.
3. zipalign要在sign之后，逆序后sign会undo the alignment.
	
### 6. Image Tools

#### 1. Draw 9-patch
#### 2. etc1tool

## 二、 Platform Tools

### 1. bmgr
### 2. logcat