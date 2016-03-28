title: Android-DDMS
categories:
  - Android
tags:
  - Android
  - DDMS
date: 2016-01-18 16:33:10
---
DDMS的介绍及各个tool的使用。

## 一、 DDMS
Dalvik Debug Monitor Server (DDMS)。

* every application runs in its own process,
* every process runs in its own virtual machine (VM).
* every VM exposes a unique port that a debugger can attach to.


DDMS assigns a debugging port to each VM on the device. 8600 for the first VM, the next on 8601.    
The DDMS "base port" (8700, by default). The base port is a port forwarder, which can accept VM traffic from any debugging port and forward it to the debugger on port 8700. 


1. You don't always have to restart your app to debug it. To debug an app that you're already running:    
![](https://github.com/ccSun/hexoBlogOnGitHub/blob/master/source/_posts/android-ddms/ddms_connect.png?raw=true)
2. 分析页面的布局结构，树状结构看到布局及item id。Dump view hierarchy from ui automator.    
![](https://github.com/ccSun/hexoBlogOnGitHub/blob/master/source/_posts/android-ddms/view_hierarchy.png?raw=true)