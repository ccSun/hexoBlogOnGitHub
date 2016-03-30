title: Android-Build-Process
categories:
	- Android
tags:
	- Android
	- BuildProcess
date: 2016-03-17 15:35:48
---
android app的编译过程。

## 一、 Build Process

![Build Process](https://github.com/ccSun/hexoBlogOnGitHub/blob/master/source/_posts/android-build-process/build_process.png?raw=true)

## 二、 Building in Debug Mode

In debug mode, the build tools automatically sign your application with a debug key and optimize the package with zipalign.

On Windows platforms, type this command:

```
gradlew.bat assembleDebug
```

On Mac OS and Linux platforms, type these commands:

```
$ chmod +x gradlew
$ ./gradlew assembleDebug
```

***Output:***

* APK for the app module is located in app/build/outputs/apk/
* AAR for any lib modules is located in lib/build/outputs/libs/

## 三、 Building in Release Mode

There are two approaches to building in release mode: build an unsigned package in release mode and then manually sign and align the package, or allow the build script to sign and align the package for you.

### 1. Build unsigned and aligned

```
apply plugin: 'com.android.application'

android {
    compileSdkVersion 19
    buildToolsVersion "19.0.0"
    // buildToolsVersion should be higher or equal to compileSdkVersion

	// values in defaultConfig override those in the manifest file
    defaultConfig {
        applicationId "com.example.my.app" //发布pro和free时用
        minSdkVersion 8
        targetSdkVersion 19 
        versionCode 1
        versionName "1.0"
    }
    
    // 编译时将会同时编译两个apk出来
    // 将会覆盖defaultConfig里的值
    // 创建不同的flavor代码，目录结构src/main和src/free,子目录相同
    productFlavors {
        pro {
            applicationId = "com.example.my.pkg.pro"
            versionName "1.0-pro"

        }
        free {
            applicationId = "com.example.my.pkg.free"
            versionName "1.0-free"
        }
    }

	// 执行编译命令的时候gradlew assembleCustom编译custom
	// gradlew assemblerelease编译release
	// 都将编译出pro和free两个包
	// 编译custom的时候，会在包名后再加一个后缀
    buildTypes {
        release {
            minifyEnabled true
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rulse.pro'
        }
       
        // debug，alpha， beta
        
        custom {
            applicationIdSuffix ".custom"
        }
    }
}

dependencies {
    compile project(":lib")
    compile 'com.android.support:appcompat-v7:19.0.1'
    compile fileTree(dir: 'libs', include: ['*.jar'])
}
```

1. 为何要用applicationId？ manifest里的packagename是可以唯一标示apk的。packagename也用于很多资源文件的包头，包括R.java。如果同时发布一个pro和一个free版本，代码里引用的又都是com.xxx.R;再发布的时候两个id相同的是不能发布的。所以用packagename来确定包头，用applicationId来区分不同的app。对应的debug版本添加applicationIdSuffix ".debug"。

2. Source directories
To build each version of your app, the build system combines source code and resources from:
	* src/main/ - the main source directory (the default configuration common to all variants)
	* src/<buildType>/ - the source directory
	* src/<productFlavor>/ - the source directory
The build type and product flavor source directories are optional.    
For projects that do not define any flavors, the build system uses the defaultConfig settings:
	* src/main/ (default configuration)
	* src/release/ (build type)
	* src/debug/ (build type)

3. For projects that define a set of product flavors, the build system merges the build type, product flavor and main source directories. also merges all the manifests into a single manifest. The manifest merge priority from lowest to highest is libraries/dependencies -> main src -> productFlavor -> buildType. Resources with the same name also applies for this priority sequence.

4. manifest合并修改语法：http://developer.android.com/tools/building/manifest-merge.html#markers-selectors


***Output***    
This creates your Android application .apk file inside the module build/ directory, named <your_module_name>-release.apk. This .apk file has been signed with the private key specified in build.gradle file and aligned with zipalign. It's ready for installation and distribution.


### 2. Build signed and aligned


## 四、 Apps Over 65k Methods

1. build error:
	* Earlier versions of the build system report this error as follows
	
		```
		Conversion to Dalvik format failed:
Unable to execute dex: method ID not in [0, 0xffff]: 65536
		```
		
	* More recent versions of the Android build system display a different error
	
		```
		trouble writing output:
Too many field references: 131000; max is 65536.
You may try using --multi-dex option.
		```
一个dalvik里能引用的方法数上限65536个。


2. Android application (APK) files contain executable bytecode files in the form of Dalvik Executable (DEX) files；The Dalvik Executable specification ***limits the total number of methods that can be referenced within a single DEX file to 65,536***, including Android framework methods, library methods, and methods in your own code. ***Getting past this limit requires that you configure your app build process to generate more than one DEX file***, known as a multidex configuration. 

3. Solution:
http://developer.android.com/tools/building/multidex.html#about