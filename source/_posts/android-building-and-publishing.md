title: android-building-and-publishing
categories:
  - Android
tags:
  - Android
  - Build
  - Gradle
date: 2015-11-25 15:46:41
---
Android app build and publish processes.


## 一、 Signing Types
1. Signing in Debug Mode    
Android Studio signs your app in debug mode automatically when you run or debug your project from the IDE.The debug keystore is located in $HOME/.android/debug.keystore, and is created if not present.

2. Signing in Release Mode    

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

## 二、 Signing Considerations

1. ***App upgrade***    
The system allows the update if the certificates match. If you sign the new version with a different certificate, you must assign a different package name to the application—in this case, the user installs the new version as a completely new application.
2. ***App modularity***    
Android ***allows apps signed by the same certificate to run in the same process***, if the applications so requests, so that the system treats them as a single application. In this way you can deploy your app in modules, and users can update each of the modules independently.
3. ***Code/data***    
By signing multiple apps with the same certificate and using signature-based permissions checks, your apps can share code and data in a secure manner.

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