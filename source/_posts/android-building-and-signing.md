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

## 二、 Building Mode
### 1. Build In Debug Mode

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

### 2、 Building in Release Mode

There are two approaches to building in release mode: build an unsigned package in release mode and then manually sign and align the package, or allow the build script to sign and align the package for you.

##### 1. Build unsigned and aligned

```
// 导入gradle的编译android的控件
apply plugin: 'com.android.application'

android {
    compileSdkVersion 19 // sdk代码版本，高版本会提示遗弃的方法
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
    
    signingConfigs {
        releaseMy {
            storeFile file("myreleasekey.keystore")
            storePassword "password"
            keyAlias "MyReleaseKey"
            keyPassword "password"
            
            // 从环境变量获取            
            storePassword System.getenv("KSTOREPWD")
            keyPassword System.getenv("KEYPWD")
            
            // 从console获取
            storePassword System.console().readLine("\nKeystore password: ")
            keyPassword System.console().readLine("\nKey password: ")
        }
    }
    
    // 编译不同的版本app，比如free和paid版本，他们有common模块也有不同的模块
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

	// 开发周期不同阶段编译，比如debug时用debug keystore签名，release包可以shrink并用release keystore签名
	// 至少定义一个buildtype，Studio默认创建debug和release
	// 执行编译命令的时候gradlew assembleCustom编译custom
	// gradlew assemblerelease编译release
	// 都将编译出pro和free两个包
	// 编译custom的时候，会在包名后再加一个后缀
    buildTypes {
        release {
            minifyEnabled true
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rulse.pro'
            
            signingConfig signingConfigs.releaseMy // 使用releaseMy的配置
            
            
            resConfigs "en", "fr" //只使用en和fr的language
        }
        
        
        // 也可以是debug，alpha， beta
       	jnidebug {

            // debug配置的一份copy，接着下面的定义会覆盖debug里的参数
            initWith debug

            applicationIdSuffix ".jnidebug"
            jniDebuggable true
        }
        
    }
    
    dependencies {
    // The 'compile' configuration tells Gradle to add the dependency to the
    // compilation classpath and include it in the final package.

    // Dependency on the "mylibrary" module from this project
    compile project(":mylibrary")

    // Remote binary dependency
    compile 'com.android.support:appcompat-v7:23.4.0'

    // Local binary dependency
    compile fileTree(dir: 'libs', include: ['*.jar'])
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


##### 2. Build signed and aligned


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

## 三、 Signing App Manually

1. Generate a private key using keytool

		$ keytool -genkey -v -keystore my-release-key.keystore -alias alias_name -keyalg RSA -keysize 2048 -validity 10000

2. Sign your app with your private key using jarsigner

		$ jarsigner -verbose -sigalg SHA1withRSA -digestalg SHA1 -keystore my-release-key.keystore my_application.apk alias_name

3. Verify that your APK is signed

		$ jarsigner -verify -verbose -certs my_application.apk
		
4. Align the final APK package using zipalign

		$ zipalign -v 4 your_project_name-unaligned.apk your_project_name.apk
		
zipalign ensures that all uncompressed data starts with a particular byte alignment relative to the start of the file, which reduces the amount of RAM consumed by an app.对齐资源文件、数据文件，在访问的时候可以节省内存，加快效率。


## 四、 Signing Considerations

1. ***App upgrade***    
The system allows the update if the certificates match. If you sign the new version with a different certificate, you must assign a different package name to the application—in this case, the user installs the new version as a completely new application.
2. ***App modularity***    
Android ***allows apps signed by the same certificate to run in the same process***, if the applications so requests, so that the system treats them as a single application. In this way you can deploy your app in modules, and users can update each of the modules independently.
3. ***Code/data***    
By signing multiple apps with the same certificate and using signature-based permissions checks, your apps can share code and data in a secure manner.



## 五、 Merge Manifest File

### 1. Priority

High -> Low:    

1. buildTypes manifest file
2. productFlavors manifest file
3. src/main
4. dependencies and libraries manifest file

### 2. Merge Rules

| High Priority 	| Low Priority 	| merge Result |
|-|-|-|
| 无值				| 无值			| 无值		|
| 无值				| default		| default	|
| 无值				| non-def		| non-def	|
||||
| def				| 无值			| def		|
| def				| default		| default	|
| def				| non-def		| non-def	| 
||||
| non-def			| 无值			| non-def	|
| non-def			| default		| non-def	|
| non-def			| non-def		| Merge if settings match, otherwise causes conflict error.	|

***Note***    

1. manifest的元素只合并该元素的子元素；
2. intent-filter只会添加到common父节点，不会合并；
3. library的minSdkVersion大于高优先级的设置，则无法合并；除非手动指定overrideLibrary；

### 3. Merge Gramma

可以解决library的minSdkVersion大于高优先级的manifest的设定时候无法合并的问题；

##### 1. Keywords

1. merge 默认合并
2. replace 高优先级替换低优先级
3. strict 设定的属性值必须相同，否则build报错
4. merge-only 只合并低优先级
5. remove 移除指定节点
6. remove-all 移除相同节点下所有低优先级属性


```
<application
	android:icon="@drawable/icon"
	android:label="@string/app_name"
	tools:replace="icon, label">
	
// 覆盖低优先级的android:targetSdkVersion，android:minSdkVersion
<uses-sdk android:targetSdkVersion="22" android:minSdkVersion="2"
tools:overrideLibrary="com.example.lib1, com.example.lib2"/>
     
// 移除com.example.lib1的这个权限
<permission
android:name="permissionOne"
tools:node="remove"
tools:selector="com.example.lib1">
```

##### 2. 注入变量applicationId

Manifest entry:

```
<activity android:name=".Main">
    <intent-filter>
        <action android:name="${applicationId}.foo"></action>
    </intent-filter></activity>
```

Gradle build file:

```
android {
    compileSdkVersion 22
    buildToolsVersion "22.0.1"

    productFlavors {
        flavor1 {
            applicationId = "com.mycompany.myapplication.productFlavor1"
        }
    }

```

##### 3. 注入其他属性

Manifest entry:    

```
<activity android:name=".MainActivity" android:label="${activityLabel}" >
```

Gradle build file:

```
android {
    defaultConfig {
        manifestPlaceholders = [ activityLabel:"defaultName"]
    }
    productFlavors {
        free {
        }
        pro {
            manifestPlaceholders = [ activityLabel:"proName" ]
        }
    }
```

## 六、 Improve build times by configuring DEX resources

1. build.gradle 配置Dex编译项

	```
	android {
	  ...
	  dexOptions {
	    maxProcessCount 4 // this is the default value
	    javaMaxHeapSize "2g" // 2m 2k单位
	  }
	}
	```

	1. maxProcessCount：设置可以同时启动的Dex Process数量；./gradlew --stop可以停掉以启动的gradle进程；
	2. javaMaxHeapSize：Dex操作可以分配的最大内存；    
	
	***Note***    
	但是并不是分配的越大越好，越大反而会越慢。要多次设置调试观察build时间。

2. gradle.properties 设置Dex-in-process

	```
	org.gradle.jvmargs = -Xmx2048m // def 2G，如果设置了javaMaxHeapSize，这里的值设置为javaMaxHeapSize＋1024M
	```

	1. Incremental Java compilation：默认开启，只变异变动的java files；
	2. Dex-in-process可以不单单是加快incremental build过程，实际上可以加快整个build过程


## 七、 Apps Over 64k Methods

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

1. 一个dalvik里能引用的方法数量上限65536（65,536＝64 X 1024，又称64k reference limit）个。Android application (APK) files contain executable bytecode files in the form of Dalvik Executable (DEX) files；The Dalvik Executable specification ***limits the total number of methods that can be referenced within a single DEX file to 65,536***, including Android framework methods, library methods, and methods in your own code. ***Getting past this limit requires that you configure your app build process to generate more than one DEX file***, known as a multidex configuration. 

2. Solution:    
	
	1. 对引入的大dependency进行简化，去掉不必要的代码
	2. 用progurad移除不必要的代码

3. 打开multi-dex:

	1. build.gradle 
	
		```
        android {
            compileSdkVersion 21
            buildToolsVersion "21.1.0"

            defaultConfig {
            ...
            minSdkVersion 14
            targetSdkVersion 21
            ...

            // Enabling multidex support.
            multiDexEnabled true
            }
            ...
        }

        dependencies {
            compile 'com.android.support:multidex:1.0.0'
        }
		```
	
	2. manifest
	
		```
		<manifest xmlns:android="http://schemas.android.com/apk/res/android"
            package="com.example.android.multidex.myapplication">
            <application
                ...
          		 android:name="android.support.multidex.MultiDexApplication">
                ...
            </application>
        </manifest>
		```

***Note:***
1. 5.0 API21 以前，是dalvik，默认单dex，会有64k reference limit问题；
2. 5.0及以后，是aot，默认就支持多dex。

4. Multi-dex局限性

会有ANR，Crash等问题。

5. 优化Multi-dex编译

打开multi-dex会增加编译时间，因为build process需要决定哪些方法放在第一个dex，哪些在第二个。通过多个flavor，来调整一下在debug和发布生产包时候的时间。

```
android {
    productFlavors {
        // Define separate dev and prod product flavors.
        dev {
        	// api 21 使用ART，能更快的编译multi-dex
            // dev utilizes minSDKVersion = 21 to allow the Android gradle plugin
            // to pre-dex each module and produce an APK that can be tested on
            // Android Lollipop without time consuming dex merging processes.
            minSdkVersion 21
        }
        prod {
            // The actual minSdkVersion for the application.
            minSdkVersion 14
        }
    }
}
```
此时设置buildType为devDebug，那么在debug时可以节省很多时间。（Studio左侧弹出设置窗口，或者在Build－>Select Build Variant弹出）

6. [Test Multi-dex](https://developer.android.com/studio/build/multidex.html#testing)


## 八、 Install And Execute

1. APK打包的是dex文件。
2. 安装分为两种情况：

	1. Dalvik＋JIT：Android 5.0 API 21以前的版本，安装时JIT解析dex文件为odex，dex和odex其实都是Dalvik可以解析的字节码，Dalvik在apk运行时把Dalvik字节码解析为本机机器码执行。解析成odex，运行时dalvik不必每次都从apk中提取dex，速度加快了。但是odex因为字节对齐会比dex扩大1～4倍。
	2. ART+AOT：Android 5.0 API 21及以后的版本，安装时AOT解析dex文件为aot文件，但是命名为odex文件。这里的odex文件本质是elf文件，其中都是本机机器码。
	
3. JIT: just-in-time
4. AOT: ahead-of-time，类似于C语言编译时便确定了可执行环境的机器码。

## 九、 Instant Run

### 1. Require

1. Gradle version 2.0.0 or higher
2. minSdkVersion to 15 or higher，最优性能是minSdkVersion 21 Android 5.0及以上。因为在打开multi-dex的时候，instant run有可能会被关闭(参考[Multidex support prior to Android 5.0](https://developer.android.com/studio/build/multidex.html))。

### 2. 4 Level

1. hot swap: 只修改了method时最快，可以不重启activity，但是studio会帮你重启
2. warm swap: 改变或者删除资源文件，需要重启activity
3. cold swap: API 21以上需要重启app，以下需要build一个新的apk
4. new deploy: 只要改变了manifest，都需要重新编译apk。所以在build.gradle的debug type下不要动态update manifest文件

### 3. Configure Instatn Run

1. Open the Settings or Preferences dialog.
2. Navigate to Build, Execution, Deployment > Instant Run and click Update Project,