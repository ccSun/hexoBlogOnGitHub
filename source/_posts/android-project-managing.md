title: Android-Project-Managing
categories:
  - Android
tags:
  - Android
  
date: 2016-02-24 11:07:14
---

Android Studio项目目录介绍

## 1. IDE Setting

1. **.idea**
2. **.iml** Module file created by the IntelliJ IDEA to store module information.

## 2. Gradle 相关
1. **gradle** gradler wrapper
2. **.gradle**
3. **gradle.properties** project-wide gradle setting, such as MaxPermSize.
4. **local.properties** computer-specific properties.
5. **build.gradle** Customizable properties for the build system. Set keystore, keyalias.
6. **setting.gradle** Specifies the sub-modules to build.
7. **gradlew** gradle startup script for Unix.
8. **gradlew.bat** gradle startup script for Windows.

## 3. Git
1. .gitignore

## 4. 项目相关
1. **app** module directories and files
2. **build** build outputs for all project modules.

### app目录分解

1. **build**
2. **libs**
	1. Dependencies添加lib，module和本地jar lib；
	2. Resource conflicts: select the resource from the application, or the library with highest priority, and discard the other resource.
	3. cant export a library module to a jar file;
	4. can include a JAR lib;
	5. can depend on an external JAR lib;
	
		```
    	<uses-library   	
      		android:name="string"   
      		android:required=["true" | "false"] 
      		/>
		```
	6. lib cannot include raw assets;
	7. lib platform version要等于或低于app version;
3. **src**

#### src目录分解

1. **proguard-rules.pro**
2. **.iml**
3. **build.gradle**
4. **.gitignore**

1. **androidTest**
2. **main/AndroidManifest**   
    
    ```
		<uses-adk
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
5. **main/gen**
6. **main/assets**
7. **main/res**
	1. anim	
	2. **drawable** For bitmap files (PNG, JPEG, or GIF), 9-Patch image files, and XML files that describe Drawable shapes or Drawable objects that contain multiple states (normal, pressed, or focused).
	3. **mipmap** For app launcher icons. 
	4. color
	5. layout
	6. raw
	7. values
		string.xml, color.xml, dimens.xml, styles.xml




