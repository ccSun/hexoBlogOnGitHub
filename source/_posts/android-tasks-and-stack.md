title: Android-Tasks-And-Stack
categories:
  - Android
tags:
  - Android
  - Tasks
  - Stack
date: 2016-03-15 18:26:19
---
收录Activity栈管理和操作的相关内容。
## 一、 Managing Tasks

### 1. 两种栈管理方式：    

#### 1. <activity> attributes
    
* taskAffinity
* launchMode
* allowTaskReparenting
* clearTaskOnLaunch
* alwaysRetainTaskState
* finishOnTaskLaunch    

#### 2. intent flags

* FLAG_ACTIVITY_NEW_TASK
* FLAG_ACTIVITY_CLEAR_TOP
* FLAG_ACTIVITY_SINGLE_TOP

## 二、 Defining launch modes
    
As such, if Activity A starts Activity B, ***Activity B can define in its manifest how it should associate with the current task*** (if at all) and ***Activity A can also request how Activity B should associate with current task***. If both activities define how Activity B should associate with a task, then ***Activity A's request (as defined in the intent) is honored over Activity B's request (as defined in its manifest)***.    
    
Some launch modes available for the manifest file are not available as flags for an intent and, likewise, some launch modes available as flags for an intent cannot be defined in the manifest.

### 1. Using the manifest file

1. ***standard***   
The activity can be instantiated multiple times, each instance can belong to different tasks, and one task can have multiple instances.
如果是App A启动App B里的ActB(standard),那么ActB会在App A的栈里。

2. ***singleTop***
If an instance of the activity already exists at the top of the current task, the system routes the intent to that instance through a call to its ***onNewIntent()*** method, rather than creating a new instance of the activity. 如果是App A启动App B里的ActB(singleTop),那么ActB会在App A的栈里。

3. ***singleTask***    
 However, if an instance of the activity(singleTask) already exists in a separate task, the system routes the intent to the existing instance through a call to its ***onNewIntent()*** method, rather than creating a new instance.     
 ActA启动ActB(singleTask),两个在同一个栈中。但ActB不是root元素(这是实验结果，但实验与google developer网站不符)。假设App A里栈 ActA->ActB->ActC->actD...; 在App Z里ActZ启动ActB，则将跳转App A的栈中的ActB，并且ActC、ActD等出栈，此时点击返回键直接在App A的栈中返回ActA，再点击返回键，将返回App Z的ActZ。

4. ***singleInstance***
The system doesn't launch any other activities into the task holding the instance. 不论是从App A还是App B启动该activity，有且仅有一个栈实例。

### 2. Using Intent flags

1. ***FLAG_ACTIVITY_NEW_TASK***    
same as singltTask.

2. ***FLAG_ACTIVITY_SINGLE_TOP***    
same as singleTop.

3. ***FLAG_ACTIVITY_CLEAR_TOP***   
如果栈中已有对应被启动的Activity，那么该Activity之上的所有Act出栈。如果栈中没有，那么新创建一个实例。
App B用CLEAR_TOP启动App A中的某一个Activity，这个activity将会在App B的栈中进行操作，如clear top或者new instance。

4. to be continued

### 三、 Handling affinities
The affinity indicates which task an activity prefers to belong to. By default, all the activities from the same application have an affinity for each other.

—.—||| 实在烦，看不下去，总结不出来啊
http://developer.android.com/guide/components/tasks-and-back-stack.html#ManagingTasks

### 四、 Clearing the back stack
If the user leaves a task for a long time, the system clears the task of all activities except the root activity. When the user returns to the task again, only the root activity is restored.

There are some activity attributes that you can use to modify this behavior:   
  
1. ***alwaysRetainTaskState***     
The task retains all activities in its stack even after a long period.    

2. ***clearTaskOnLaunch***
If this attribute is set to "true" in the root activity of a task, the stack is cleared down to the root activity whenever the user leaves the task and returns to it. In other words, it's the opposite of alwaysRetainTaskState. 

3. ***finishOnTaskLaunch***    
This attribute is like clearTaskOnLaunch, but ***it operates on a single activity, not an entire task***. It can also cause any activity to go away, including the root activity. When it's set to "true", the activity remains part of the task only for the current session. If the user leaves and then returns to the task, it is no longer present.

### 五、 Starting a task

—.—||| 鬼哦 没卵用