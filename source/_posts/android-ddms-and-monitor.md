title: Android-DDMS-And-AndroidMonitor
categories:
  - Android
tags:
  - Optimization
  - DDMS
  - Monitor
date: 2016-01-18 16:33:10
---
DDMS的介绍及各个tool的使用。本文介绍内容结合Android-Performance-Profiling。

## 一、 DDMS
Dalvik Debug Monitor Server (DDMS)。

* every application runs in its own process,
* every process runs in its own virtual machine (VM).
* every VM exposes a unique port that a debugger can attach to.


DDMS assigns a debugging port to each VM on the device. 8600 for the first VM, the next on 8601.    
The DDMS "base port" (8700, by default). The base port is a port forwarder, which can accept VM traffic from any debugging port and forward it to the debugger on port 8700. 


1. You don't always have to restart your app to debug it. To debug an app that you're already running:    
![](https://github.com/ccSun/hexoBlogOnGitHub/blob/master/source/_posts/android-ddms-and-monitor/ddms_connect.png?raw=true)
2. 分析页面的布局结构，树状结构看到布局及item id。Dump view hierarchy from ui automator.    
![](https://github.com/ccSun/hexoBlogOnGitHub/blob/master/source/_posts/android-ddms-and-monitor/view_hierarchy.png?raw=true)

### 1. Viewing heap usage for a process
### 2. Tracking memory allocation of objects
### 3. Working with an emulator or device's file system
### 4. Examining thread information
### 5. Starting method profiling
### 6. Using the Network Traffic tool
### 7. Emulating phone operations and location
1. Changing network state, speed, and latency
2. Spoofing calls or SMS text messages
3. Setting the location of the phone

## 二、 AndroidMonitor
 1. Switching between Devices and Apps 
 2. Taking a Screen Capture of the Device 
 3. Recording a Video from the Screen 
 4. Examining System Information     
 ![](https://github.com/ccSun/hexoBlogOnGitHub/blob/master/source/_posts/android-ddms-and-monitor/system_info.png?raw=true)
    * Activity Manager State - dumpsys activity
    * Package Information - dumpsys package
    * Memory Usage - dumpsys meminfo
    * Memory Use Over Time - dumpsys procstats
    * Graphics State - dumpsys gfxinfo
    
### 1. Logcat
### 2. Memory Monitor
参考Android-Performance-Profiling
### 3. CPU Monitor
参考Android-Performance-Profiling
### 4. GPU Monitor
参考Android-Performance-Profiling
### 5. Network Monitor