title: Android-Shrink-Code-And-Resources
categories:
  - Android
tags:
  - Optimization
date: 2015-09-15 22:28:02
---
Progurad可以做到code shrinking，减少包的大小，也可以避免64k references limit的问题。与此同时我们也可以打开resource shrinking。

## 一、Shrink Code with ProGuard

1. 优化方式：移除二进制bytecode中未使用的classes/fields/methods;删除bytecode中的debuging info(文件名，行数，方法名变量名等)，模糊重命名类、变量和方法；也可以帮助比避免64k reference limit的问题。
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
	***Note***    
	如果打开了code shrinking，会减慢build速度。所以debug时不要开启。但是最终打包的时候一定要开启。

4. progurad file:

	* ***proguard-android.txt*** : the default ProGuard settings from the Android SDK tools/proguard/ folder
	* ***proguard-android-optimize.txt*** : file is also available in this Android SDK folder with the same rules but with optimizations enabled. 不建议使用，并不是所有的优化都能在所有的dalvik上正确运行。
	* ***proguard-rules.pro*** : file is at root of the module for customing ProGuard rules specific to the current module. 

	如下的设定，会使用默认的proguard-android.txt和指定的proguard-rules.pro，proguard-rules-new.pro，other-rules.pro。

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

6. 可以保留部分代码不混淆，参考语法。 

6. Decoding Obfuscated Stack Traces

编译时产生的信息文件路径在： ***<module-name>/build/outputs/mapping/release/***，包括有：

* dump.txt class文件的结构
* mapping.txt 混淆的mapping 文件
* seeds.txt 没混淆的文件
* usage.txt 被删除的code


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

## 二、Shrink Resources

1. 打开resources shrinking，必须要先打开code shrinking。当code shrinking删除代码时，resources shrinking可以移除没有被引用的代码。这样，尤其在导入library时，需要先移除library中未使用的代码，这样资源文件不再被library引用，才能移除。

	***Note***
	
	Resources shrinking不能移除values/下面的资源文件。


2. Enable shrinkResources true in build.gradle.

```
android {
    ...
    buildTypes {
        release {
            shrinkResources true
            minifyEnabled true
            proguardFiles getDefaultProguardFile('proguard-android.txt'),
                    'proguard-rules.pro'
        }
    }
}
```   

3. 保留某些资源文件不移除，参考语法。
4. 移除不必要的values下的设置。

一个lib中可能有多个language，但是只期望两种。如下则会舍去其他的language。

```
android {
    defaultConfig {
        ...
        resConfigs "en", "fr"
    }
}
```

5. 资源文件merge优先顺序，由低到高（合并manifest 的顺序也是这个优先级）：

Dependencies → Main → Build flavor → Build type

