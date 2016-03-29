title: Android-Tools
categories:
  - Android
tags:
  - Android
  - Tools
  - Optimization
date: 2016-01-29 09:40:09
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

1. ~/sdk/build-tools/xxxx/aapt dump badging xxx.apk 查看包的基本信息，如packageid，version，permission等
##### 3. ProGuard

1. 优化方式：移除未使用代码，模糊重命名类、变量和方法；
2. 好处：更小的apk包，并且很难被反破解；
3. Enable ***minifyEnabled*** property in the build.gradle
 
    ```
    android {
       ...
     
        buildTypes {
            release {
                minifyEnabled true
                proguardFiles getDefaultProguardFile('proguard-android.txt'),
                'proguard-rules.pro'
            }
        }
      }
    ```

4. progurad file:
	* ***proguard-android.txt*** : the default ProGuard settings from the Android SDK tools/proguard/ folder
	* ***proguard-android-optimize.txt*** : file is also available in this Android SDK folder with the same rules but with optimizations enabled. 不建议使用，并不是所有的优化都能在所有的dalvik上正确运行。
	* ***proguard-rules.pro*** : file is at root of the module for customing ProGuard rules specific to the current module. 
***下面是没看懂的一段。。。。***
> You can also add ProGuard files to the getDefaultProguardFile directive for all release builds or as part of the productFlavor settings in the build.gradle file to customize the settings applied to build variants. This example adds the proguard-rules-new.pro to the proguardFiles directive and the other-rules.pro file to the flavor2 product flavor. 

    ```
        android {
       ...
     
        buildTypes {
            release {
                minifyEnabled true
                proguardFiles getDefaultProguardFile('proguard-android.txt'),
                'proguard-rules.pro', 'proguard-rules-new.pro'
            }
        }
     
       productFlavors {
            flavor1 {
            }
            flavor2 {
                proguardFile 'other-rules.pro'
            }
        }
     }
    ```

5. Configuring ProGuard
ProGuard might remove code that it thinks is not used, but your application actually needs. Some examples include:
	* a class that is referenced only in the AndroidManifest.xml file
	* a method called from JNI
	* dynamically referenced fields and methods    
***Proguard Manual :*** https://stuff.mit.edu/afs/sipb/project/android/sdk/android-sdk-linux/tools/proguard/docs/index.html#manual/introduction.html    
***Proguard Troubleshooting :*** https://stuff.mit.edu/afs/sipb/project/android/sdk/android-sdk-linux/tools/proguard/docs/index.html#manual/troubleshooting.html    

6. Decoding Obfuscated Stack Traces
Whenever ProGuard runs, it outputs a mapping.txt file, which shows you the original class, method, and field names mapped to their obfuscated names.

	```
	retrace.bat|retrace.sh [-verbose] mapping.txt [<stacktrace_file>]
	```
You can fix this by :

	```
	-keep public class <MyClass>
	```
7. Debugging considerations for published applications
Save the mapping.txt file for every release that you publish to your users. A project's mapping.txt file is overwritten every time you do a release build.


### 3. zipalign

1. 优化方式：保证apk内未压缩的资源文件，比如图片和二进制流文件四子节边界对其；
2. 好处：运行时节省内存。This allows all portions to be accessed directly with mmap() even if they contain binary data with alignment restrictions.
3. zipalign要在sign之后，逆序后sign会undo the alignment.
	
### 5. Image Tools

#### 1. Draw 9-patch
#### 2. etc1tool

## 二、 Platform Tools

### 1. bmgr
### 2. logcat