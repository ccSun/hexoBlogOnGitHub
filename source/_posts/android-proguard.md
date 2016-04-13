title: Android-Proguard
categories:
  - Android
tags:
  - Tools
date: 2015-09-15 22:28:02
---
Proguard的开启和相关语法。


1. 优化方式：移除二进制bytecode中未使用的classes/fields/methods;删除bytecode中的debuging info(文件名，行数，方法名变量名等)，模糊重命名类、变量和方法；
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

