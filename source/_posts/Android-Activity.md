title: Android-Activity
categories:
  - Android
tags:
  - Android
  - Acivity
  - 内存优化
  - 内存溢出
date: 2016-03-11 14:59:37
---

## 一、 Acvitity生命周期

![Acvitity Lifecycle](https://github.com/ccSun/ccsun.github.io/blob/master/2016/03/11/android-Activity/activity_lifecycle.png?raw=true)


### 1. Starting an Activity

1. you might call finish() from within onCreate() to destroy the activity. In this case, the system immediately calls onDestroy() without calling any of the other lifecycle methods.

### 2. Pause Your Activity

1. You should usually use the onPause() callback to:
	 * Stop animations or other ongoing actions that could **consume CPU**.
	 * **Commit unsaved changes**, but only if users expect such changes to be permanently saved when they leave (such as a draft email).保存永久的数据要放到onDestroy();因为保存draft需要被用户看到，所以放到onPause();
	 * **Release system resources**, such as broadcast receivers, handles to sensors (like GPS), or any resources that may affect battery life while your activity is paused and the user does not need them.	 

2. **avoid performing CPU-intensive work during onPause()**, such as writing to a database, because **it can slow the visible transition to the next activity** (you should instead **perform heavy-load shutdown operations during onStop()**).

### 3. Resume Your Activity

1. you should implement onResume() to initialize components that you release during onPause() and perform any other initializations that must occur each time the activity enters the Resumed state (such as begin animations and initialize components only used while the activity has user focus).

### 4. Stop Your Activity

1. Once your activity is stopped, **the system might destroy the instance if it needs to recover system memory**. In extreme cases, the system might **simply kill your app process without calling the activity's final onDestroy()** callback, so it's important you **use onStop() to release resources that might leak memoy..*

2. **Even if the system destroys your activity while it's stopped, it still retains the state of the View objects** (such as text in an EditText) in a Bundle (a blob of key-value pairs) and restores them if the user navigates back to the same instance of the activity

### 5. Start/Restart Your Activity

1. the user might have been away from your app for a long time before coming back it, the **onStart()** method is a good place to **verify that required system features are enabled.**

### 6. Recreating an Activity

1. When your activity is **destroyed because the user presses Back or the activity finishes itself, the system's concept of that Activity instance is gone forever** because the behavior indicates the activity is no longer needed. However, if the system **destroys the activity due to system constraints (rather than normal app behavior), then although the actual Activity instance is gone, the system remembers that it existed.** 只要是被系统销毁的activity，系统都会保存Activity的状态，这是系统的责任。
2. Your activity will be **destroyed and recreated each time the user rotates the screen.**
3. If onSaveInstanceState() is called, **this method will occur before onStop().** There are no guarantees about whether it will occur before or after onPause().

### 7. Destroy an Activity
1. onPause() -> Process Killed
2. onPause() -> onStop() -> Process Killed
3. onPause() -> onStop() -> onDestroy()

## 二、 状态变化

假设在一个EditText中输入文本，

1. 按Home键，再重新返回，文本还在。生命周期onPause()->onStop();
2. 切换横屏，文本会消失，因为此时Activity会onDestroy()后，重新onCreate()；
3. Activity在onStop()状态直接被销毁了，在点击返回键返回原Activity的时候，文本还在；

### 1. Save Your Activity State

1. When onSaveInstanceState() will be called:
	1. When you press button Home; But when you press Back this func will not be called;
	2. When one acivity come in front of the current one, such as someone call in;
	3. When you press Power button;
	4. WHen you long press Power button to change to another application;
	5. WHen you rotate the screen;
	总结起来就是：当系统有可能在你不知道的情况下销毁Activity的情况下，系统会帮你调用onSaveInstanceState()给你机会保存数据。实际操作的体验是：只有切换横竖屏的时候，文本框的数据会消失，其他情况都不会消失。但是确实都会call onSaveInstanceState()。默认的实现中，系统已经默认提供实现保存ui的状态信息。

### 2. Restore Your Activity State


1. Both the onCreate() and onRestoreInstanceState() callback methods receive the same Bundle that contains the instance state information.

	* onCreate() method is called whether the system is creating a new instance of your activity or recreating a previous one, **you must check whether the state Bundle is null** before you attempt to read it. So you can restore some state data in onCreate()
	* onRestoreInstanceState(), which the system **calls after the onStart() method.** The system **calls onRestoreInstanceState() only if there is a saved state to restore, so you do not need to check whether the Bundle is null**.

2. Always call the superclass implementation of onRestoreInstanceState() so the default implementation can restore the state of the view hierarchy.


### 3. Calling sequence

#### a. onStoreInstanceState() & onRestoreInstanceState()
onStart() -> onRestoreInstanceState()    
onSaveInstanceState() -> onStop()


***There are no guarantees about whether it will occur before or after onPause().***

#### b. Activity A & B

(1) A start B:    
A onPause() -> B onCreate() -> B onStart() -> B onResume() -> A onStop()    
(2) B returen to A:    
B onPause() -> A onRestart() -> A onStart() -> A onResume() -> B onStop() -> B onDestroy()    


## 三、 Leaked Activity

1. Call finish() after startActivity() will lead A activity to a leaked activity. // 内存优化 内存溢出

    ```
		Intent intent = new Intent(A.this, B.class);
		MainActivity.this.startActivity(intent);
		MainActivity.this.finish();
	```
2. to be continued


## 四、 Activity Launch Mode

### 1. Brief Introduction
 
Activity Stack is a LIFO stack.    
一个应用程序的优先级是受最高优先级的Activity影响的。Android内存管理使用栈来决定基于Activity的应用程序的优先级,决定某个应用程序是否要终结去释放资源。 

### 2. 



