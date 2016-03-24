title: Android-Overview-Screen
categories:
  - Android
tags:
  - Android
  - Tasks
  - Stack
  - OverviewScreen
date: 2016-03-24 14:36:50
---
有关Recents列表的操作。但是并没有完全弄懂。收录地址http://developer.android.com/guide/components/recents.html

## 一、Brief Introduction

1. 就是后台任务栏，referred to as the recents screen, recent task list, or recent apps) is a system-level UI that lists recently accessed activities and tasks.
2. With the ***Android 5.0 release (API level 21), multiple instances of the same activity containing different documents may appear as tasks in the overview screen***. For example, Google Drive may have a task for each of several Google documents. Each document appears as a task in the overview screen.
3. The ***ActivityManager.AppTask*** class ***lets you manage tasks***, and the activity flags of ***the Intent*** class ***let you specify when an activity is added or removed from the overview screen***. Also, ***the <activity> attributes let you set the behavior in the manifest***.

## 二、 Adding Tasks to the Overview Screen

启用multiple tasks功能，被启动的Act的launchmode必须是standard。

### 1. Two types Diff

1. \<activity> attr:     
You can choose between ***always*** opening the document in a new task or reusing an existing task for the document
2. Intent flag:    
affords greater control over when and how a document gets opened or reopened in the overview screen

### 2. Using the Intent flag to add a task

1. When you create a new document for your activity, pass the ***FLAG_ACTIVITY_NEW_DOCUMENT*** flag in the ***addFlags()*** method of the Intent. you call the startActivity() method of the ActivityManager.AppTask class, passing to it the intent that launches the activity. ***The system treats your activity as a new task in the overview screen***. **The FLAG_ACTIVITY_NEW_DOCUMENT flag replaces the FLAG_ACTIVITY_CLEAR_WHEN_TASK_RESET flag, which is deprecated as of Android 5.0 (API level 21)**.    
***Note:*** Activities launched with the FLAG_ACTIVITY_NEW_DOCUMENT flag ***must have the android:launchMode="standard"*** attribute value (the default) set in the manifest.
2. 有FLAG_ACTIVITY_NEW_DOCUMENT时，将会新建一个task Act，但是相同的Act只有一个，之后启动的会走onNewIntent();如果想要每一次启动都是一个新的task Act，需要再addFlags(Intent.FLAG_ACTIVITY_MULTIPLE_TASK); 这样不论Act内容是否相同，都会创建新的Act task。

### 3. Using the activity attribute  "android:documentLaunchMode" to add a task

1. intoExisting    
same as FLAG_ACTIVITY_NEW_DOCUMENT.
2. always    
same as FLAG_ACTIVITY_NEW_DOCUMENT & FLAG_INTENT_MULTIPLE_TASK.
3. none    
default behavior.
4. never    
Setting this value overrides the behavior of the FLAG_ACTIVITY_NEW_DOCUMENT and FLAG_ACTIVITY_MULTIPLE_TASK flags.

***Note:*** For values other than "none" and "never" the activity must be defined with launchMode="standard".


## 三、 Removing Tasks

### 1. Using the activity attribute

1. Adding ***android:excludeFromRecents=true*** to \<activity> attr, You can always exclude a task from the overview screen entirely.
> 这个隐藏的act哪里去了。。关闭启动他的act也没有走onDestroy，达到maxRecents> 也没有onDestroy, 调用finishAndRemoveTask()也没有onDestroy
> 

2. You can set the maximum number of tasks that your app can include in the overview screen by setting the <activity> attribute ***android:maxRecents*** to an integer value.     
Default is 16; 到达上限后least recently uesed is removed但是没有onDestroy！ 最大值是50（25 on low memory devices）。最小值1。

### 2. Using the AppTask class to remove tasks

1. finishAndRemoveTask();     
> finish all activities associated with the activity.然而并没有搞懂有什么意思。难道类似于 system.exit(0)?
2. intent add FLAG_ACTIVITY_RETAIN_IN_RECENTS
即使Act已经OnDestroy了，但是在recents列表里仍有显示，点击后会重新onCreate。