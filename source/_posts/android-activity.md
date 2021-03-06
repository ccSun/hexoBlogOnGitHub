title: Android-Activity
categories:
  - Android
tags:
  - Acivity
  - Optimization
date: 2015-10-04 14:59:37
---
收录所有与Activity相关的内容。如生命周期，状态保存，内存溢出，启动模式等。

## 一、 Acvitity生命周期

![Acvitity Lifecycle](https://github.com/ccSun/hexoBlogOnGitHub/blob/master/source/_posts/android-activity/activity_lifecycle.png?raw=true)


### 1. Starting an Activity

1. you might call finish() from within onCreate() to destroy the activity. In this case, the system immediately calls onDestroy() without calling any of the other lifecycle methods.
2. For example, if your activity has a thread running in the background to download data from the network, it might create that thread in onCreate() and then stop the thread in onDestroy().

### 2. Pause Your Activity

Another activity is visible on top of this one and that activity is partially transparent or doesn't cover the entire screen.A paused activity is completely alive, but can be killed by the system in extremely low memory situations. the system can drop it from memory either by asking it to finish (calling its finish() method), or simply killing its process.

1. You should usually use the onPause() callback to:
	 * Stop animations or other ongoing actions that could **consume CPU**.
	 * **Commit unsaved changes**, but only if users expect such changes to be permanently saved when they leave (such as a draft email).保存永久的数据要放到onDestroy();因为保存draft需要被用户看到，所以放到onPause();
	 * **Release system resources**, such as broadcast receivers, handles to sensors (like GPS), or any resources that may affect battery life while your activity is paused and the user does not need them.	 

2. **avoid performing CPU-intensive work during onPause()**, such as writing to a database, because **it can slow the visible transition to the next activity** (you should instead **perform heavy-load shutdown operations during onStop()**).
3. Because this state (Resume & Pause) can transition often, the code in these two methods should be fairly lightweight in order to avoid slow transitions that make the user wait.

### 3. Resume Your Activity

1. you should implement onResume() to initialize components that you release during onPause() and perform any other initializations that must occur each time the activity enters the Resumed state (such as begin animations and initialize components only used while the activity has user focus).
2. When the activity resumes, you can reacquire the necessary resources and resume actions that were interrupted. 
3. Because this state (Resume & Pause) can transition often, the code in these two methods should be fairly lightweight in order to avoid slow transitions that make the user wait.

### 4. Stop Your Activity

The activity is completely obscured by another activity (the activity is now in the "background"). However, it is no longer visible to the user and it can be killed by the system when memory is needed elsewhere. the system can drop it from memory either by asking it to finish (calling its finish() method), or simply killing its process.

1. Once your activity is stopped, **the system might destroy the instance if it needs to recover system memory**. In extreme cases, the system might **simply kill your app process without calling the activity's final onDestroy()** callback, so it's important you **use onStop() to release resources that might leak memoy.**

2. **Even if the system destroys your activity while it's stopped, it still retains the state of the View objects** (such as text in an EditText) in a Bundle (a blob of key-value pairs) and restores them if the user navigates back to the same instance of the activity.

3. when stopped, your activity should release any large objects, such as network or database connections. onStart() & onStop() maintains resources that are needed to ***show the activity*** to the user. For example, you can register a BroadcastReceiver in onStart() to monitor changes that impact your UI, and unregister it in onStop() when the user can no longer see what you are displaying.    
***release:***
	* network
	* database
	* receiver

4. 内存不足时，会回调onLowMemory()/onTrimMemory()释放ui显示使用的图片、数组、缓存等资源。
	1. 场景：
		一般大图片、数组、缓存，如果在onStop时不释放，为了返回时更快显示；但是如果在系统资源不足时，我们要在onTrimMemory(TRIM_MEMORY_MODERATE)等级释放资源，避免进程被杀死。然后在onStart或者onResume时恢复资源；
	1. 区别：
		api<14的机器使用onLowMemory，否则都适用onTrimMemory;在onTrimMemory时最常用的level等级是ComponentCallbacks2.TRIM_MEMORY_MODERATE;    
		
		* onLowMemory() 被回调时，已经没有后台进程(优先级为background的进程)，是在最后一个后台进程被杀时调用，一般情况是low memory killer 杀进程后触发；可以自定义实现ComponentCallbacks的onLowMemory()方法:
		
			```
			Acitivity.registerComponentCallbacks(callBack);
			```
		* onTrimMemory(int) 在android 4.0之后新增，onTrimMemory被回调时，还有后台进程，触发更频繁，每次计算进程优先级时，只要满足条件，都会触发）。先调用onStop()时，不用释放ui资源，因为用户有可能返回；参数是优先级，不同的级别区分；
		
	2. 实现组件：
		* Application.onTrimMemory()
		* Activity.onTrimMemory()
		* Fragement.OnTrimMemory()
		* Service.onTrimMemory()
		* ContentProvider.OnTrimMemory()
		
	3. 参考案例：    
	
		```
		@Override
		public void onTrimMemory(int level) {
    		super.onTrimMemory(level);
    		if (level >= ComponentCallbacks2.TRIM_MEMORY_MODERATE) {
        		mAppsCustomizeTabHost.onTrimMemory();
    		}
		}
		```
	


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

When onSaveInstanceState() will be called:

1. When you press button Home; But when you press Back this func will not be called;
2. When one acivity come in front of the current one, such as someone call in;
3. When you press Power button;
4. When you long press Power button to change to another application;
5. When you rotate the screen;(onPause -> onSaveInstanceState -> onStop -> onDestroy -> onCreate -> onRestart -> onResume. 这里的调用顺序跟A启动B再返回A是有区别的。如果有onSaveInstanceState才会调用onRestoreInstanceState，所以在onRetsoreState中不用检查Bundle为空的case。)    
	
总结起来就是：当系统有可能在你不知道的情况下销毁Activity的情况下，系统会帮你调用onSaveInstanceState()给你机会保存数据。实际操作的体验是：只有切换横竖屏的时候，文本框的数据会消失，其他情况都不会消失。但是确实都会call onSaveInstanceState()。默认的实现中，系统已经默认提供实现保存ui的状态信息。前提是：The only work required of you is to provide a unique ID (with the android:id attribute) for each widget you want to save its state. If a widget does not have an ID, then the system cannot save its state.

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
2. ***to be continued***

## 四、 Start An Activity
1. implict intent
	
	```
		Intent intent = new Intent(Intent.ACTION_SEND);
		intent.putExtra(Intent.EXTRA_EMAIL, recipientArray);
		startActivity(intent);
	```
2. explict intent

	```
		Intent intent = new Intent(this, SignInActivity.class);
		startActivity(intent);
	```
3. start activity with a result

	```
        private void pickContact() {
            // Create an intent to "pick" a contact, as defined by the content provider URI
            Intent intent = new Intent(Intent.ACTION_PICK, Contacts.CONTENT_URI);
            startActivityForResult(intent, PICK_CONTACT_REQUEST);
        }

        @Override
        protected void onActivityResult(int requestCode, int resultCode, Intent data) {
            // If the request went well (OK) and the request was PICK_CONTACT_REQUEST
            if (resultCode == Activity.RESULT_OK && requestCode == PICK_CONTACT_REQUEST) {
                // ...
            }
        }
	```

## 五、 Declaring Activity <intent-filter>
1. If you intend for your application to be self-contained and not allow other applications to activate its activities, then you don't need any other intent filters. Only one activity should have the "main" action and "launcher" category.
2. However, if you want your activity to respond to implicit intents that are delivered from other applications, you must include an <intent-filter> that includes an <action> element and, optionally, a <category> element and/or a <data> element. 
1. The <action> element specifies that this is the "main" entry point to the application. 
	
	```
		<action android:name="android.intent.action.MAIN" />

	```
2. The <category> element specifies that this activity should be listed in the system's application launcher (to allow users to launch this activity).

	```
		<category android:name="android.intent.category.LAUNCHER" />

	```
***In order to receive implicit intents, you must include the CATEGORY_DEFAULT category in the intent filter***. The methods startActivity() and startActivityForResult() treat all intents as if they declared the CATEGORY_DEFAULT category. If you do not declare it in your intent filter, no implicit intents will resolve to your activity.

## 六、 Activity Launch Mode

### 1. Brief Introduction
 
Activity Stack is a LIFO stack.    
一个应用程序的优先级是受最高优先级的Activity影响的。Android内存管理使用栈来决定基于Activity的应用程序的优先级,决定某个应用程序是否要终结去释放资源。 

### 2. 启动模式
参考Android-Tasks-And-Stack。



