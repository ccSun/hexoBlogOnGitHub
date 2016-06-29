title: Android-Project-Managing
categories:
  - Android
tags:
  - 项目目录
  
date: 2015-06-03 11:07:14
---
Android项目目录基本配置和管理，以及icon及其size。
## 一、 Android Project Files

### 1. IDE Setting

1. **.idea**
2. **.iml** Module file created by the IntelliJ IDEA to store module information.

### 2. Gradle 相关
1. **gradle** gradler wrapper
2. **.gradle**
3. **gradle.properties** project-wide gradle setting, such as MaxPermSize
4. **local.properties** 与电脑环境相关的，比如sdk目录，注意不要添加到VCS里。
5. **build.gradle** Gradle的编译脚本，非常强大。
6. **setting.gradle** 指定要编译的module。
7. **gradlew** gradle startup script for Unix.
8. **gradlew.bat** gradle startup script for Windows.

### 3. Git
1. .gitignore

## 二、 Android Application Modules
1. **src** module directories and files
2. **build** build outputs for all project modules.
3. **libs**

##### src目录分解

1. **proguard-rules.pro**
1. **.iml**
1. **build.gradle**    
比如添加buildType，flavor，keystore等配置信息。
1. **.gitignore**

1. **androidTest**    
Contains code for instrumentation tests that run on an Android device. For more information, see the [Android Test documentation](https://developer.android.com/studio/test/index.html).    
1. **test**    
Contains code for local tests that run on your host JVM.    
1. **main/AndroidManifest**
   
    ```
		<uses-sdk
			android:minSdkVersion=""
			android:targetSdkVersion=""/>
		<uses-permission android:name=""/>
		<permission android:name="" android:protectionLevel=""/>
		<application>
			<activity>
				<intent-filter>
					<action
					<category
					<data
			<service
			<receiver
			<provider
	```
3. **main/java**
4. **main/jni**    
[Java Native Interface](https://developer.android.com/ndk/index.html)
5. **main/gen**    
包括studio产生的***R.java*** 和 ***AIDL（Android Interface Definition Language）生成的interface文件***。
6. **main/assets**    
Files that you save here are compiled into an .apk file as-is, and the original filename is preserved. 
7. **main/res**
	1. anim	
	2. **drawable** For bitmap files (PNG, JPEG, or GIF), 9-Patch image files, and XML files that describe Drawable shapes or Drawable objects that contain multiple states (normal, pressed, or focused).
	3. **mipmap**    
		* Launcher icons.
		* Action bar and tab icons.
		* Notification icons.
	4. color
	5. layout
	6. raw
	7. values
		string.xml, color.xml, dimens.xml, styles.xml

## 三、 Library Module

### 1. Convert an app module to a library module

Open the ***build.gradle*** set this:    

```
	apply plugin: 'com.android.library'   
```


### 2. Add library module
    
Add to ***settings.gradle***    

```
	include ':app', ':my-library-module'    
```
    
Add to ***build.gradle***    
    
```
	dependencies {
		compile project(":my-library-module")
	}
```
    
### 3. Consideration

When you build an application that depends on a library module, the SDK tools ***compile the library into a temporary JAR file*** and use it in the main module, ***then uses the result to generate the .apk***
    
***You cannot:***    

1.  cannot compile it directly to its own .apk
2.  cannot export the library module to a self-contained JAR file
3.  cannot include raw assets; 不能包含raw assets；

***You can:***
    
1.  can add Dependencies添加lib，module和本地jar lib
2.  can include a JAR into a lib module; manually edit the dependent application modules's build path and add a path to the JAR file.
3.  can include an external jar lib into a lib module;

	```
	<uses-library   	
    	android:name="string"   
    	android:required=["true" | "false"] 
    	/>
	```

***Resource conflicts***    

1. 如果一个资源文件在app和library module里同时出现，app里的优先级更高。Library module里的资源文件并不会编译进apk里
2. 如果app依赖多个lib，***dependencies***里第一个优先级最高。
3. Should use prefixes to avoid resource conflicts;

***Note***    

1. lib platform version要等于或低于app version;
2. No restriction on library module names
3. 每一个lib module都会编译成AAR文件添加到app里，而且都会有自己的R.class;

***彩蛋来了***

AAR和JAR是什么鬼？

* AAR：Android Archive，可以包含java文件，resource和manifest。
* JAR：Java Archive，只能包含java文件。


## 四、 Test Project
The src/androidTest source set may not be created for every type of available module template. If this source set is not created, you can just create it for that module.

More Info http://developer.android.com/tools/testing/index.html.