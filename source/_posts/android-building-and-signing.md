title: Android-Building-And-Signing
categories:
	- Android
tags:
	- Building
	- Signing
	- Gradle
date: 2015-06-18 15:35:48
---
Android app的编译、签名过程，及Gradle打包脚本的配置。

## 一、 Build Process

![Build Process](https://github.com/ccSun/hexoBlogOnGitHub/blob/master/source/_posts/android-building-and-signing/build_process.png?raw=true)

## 二、 Building in Debug Mode

In debug mode, the build tools automatically sign your application with a debug key and optimize the package with zipalign.The debug keystore is located in $HOME/.android/debug.keystore, and is created if not present.

On Windows platforms, type this command:

```
gradlew.bat assembleDebug  // Debug是gradle脚本的buildTypes
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

1. 为何要用applicationId？ manifest里的packagename是可以唯一标示apk的。packagename也用于很多资源文件的包头，包括R.java。如果同时发布一个pro和一个free版本，代码里引用的又都是com.xxx.R;再发布的时候两个id相同的是不能发布的。所以用packagename来确定包头，用applicationId来区分不同的app（谷歌App Play是用此来标识包的）。对应的debug版本添加applicationIdSuffix ".debug"。

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


### 2. Build signed and aligned


```
android {
    ...
    defaultConfig { ... }
    signingConfigs {
        release {
            storeFile file("myreleasekey.keystore")
            storePassword "password"
            keyAlias "MyReleaseKey"
            keyPassword "password"
        }
    }
    buildTypes {
        release {
            ...
            signingConfig signingConfigs.release
        }
    }
}
```
    
    
Including the passwords for your release key and keystore inside the build file is not a good security practice. 
    
To obtain these passwords from environment variables:

```
	storePassword System.getenv("KSTOREPWD")
	keyPassword System.getenv("KEYPWD")
```
To have the build process prompt you for these passwords if you are invoking the build from the command line:

```
	storePassword System.console().readLine("\nKeystore password: ")
	keyPassword System.console().readLine("\nKey password: ")
```

***Output***    
This creates your Android application .apk file inside the module build/ directory, named <your_module_name>-release.apk. This .apk file has been signed with the private key specified in build.gradle file and aligned with zipalign. It's ready for installation and distribution.


## 四、 Signing Considerations

1. ***App upgrade***    
The system allows the update if the certificates match. If you sign the new version with a different certificate, you must assign a different package name to the application—in this case, the user installs the new version as a completely new application.
2. ***App modularity***    
Android ***allows apps signed by the same certificate to run in the same process***, if the applications so requests, so that the system treats them as a single application. In this way you can deploy your app in modules, and users can update each of the modules independently.
3. ***Code/data***    
By signing multiple apps with the same certificate and using signature-based permissions checks, your apps can share code and data in a secure manner.

## 五、 Signing App Manually

1. Generate a private key using keytool

		$ keytool -genkey -v -keystore my-release-key.keystore -alias alias_name -keyalg RSA -keysize 2048 -validity 10000

2. Sign your app with your private key using jarsigner

		$ jarsigner -verbose -sigalg SHA1withRSA -digestalg SHA1 -keystore my-release-key.keystore my_application.apk alias_name

3. Verify that your APK is signed

		$ jarsigner -verify -verbose -certs my_application.apk
		
4. Align the final APK package using zipalign

		$ zipalign -v 4 your_project_name-unaligned.apk your_project_name.apk
		
zipalign ensures that all uncompressed data starts with a particular byte alignment relative to the start of the file, which reduces the amount of RAM consumed by an app.对齐资源文件、数据文件，在访问的时候可以节省内存，加快效率。

## 六、 Apps Over 64k Methods

1. ***Build Error:***

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

1. 一个dalvik里能引用的方法数上限65536个。Android application (APK) files contain executable bytecode files in the form of Dalvik Executable (DEX) files；The Dalvik Executable specification ***limits the total number of methods that can be referenced within a single DEX file to 65,536***, including Android framework methods, library methods, and methods in your own code. ***Getting past this limit requires that you configure your app build process to generate more than one DEX file***, known as a multidex configuration. 

2. Solution:    
http://developer.android.com/tools/building/multidex.html#about