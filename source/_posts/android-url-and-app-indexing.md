title: Android-Url-And-App-Indexing
categories:
  - Android
tags:
  - DeepLink
  - AppIndex
  - Optimization
  - Tools
date: 2015-08-13 10:32:29
---
DeepLink实现Android和H5页面跳转；App Index可以网页搜索app内部的功能。    
Android和iOS都可添加deep link以及app index：https://developers.google.com/app-indexing/introduction。


## 一、 DeepLink

### 1. Adding an Intent Filter for Deep Linking and Google Search
更多信息参考https://developers.google.com/app-indexing/introduction。内容包括给Anroid和iOS添加深层连接和App Indexing。
AndroidManifest.xml中activity，generate Deep Link。实际上是添加intent-filter.

```
<intent-filter>
    <action android:name="android.intent.action.VIEW" />

    <category android:name="android.intent.category.DEFAULT" />
    <category android:name="android.intent.category.BROWSABLE" />
    <!-- ATTENTION: This data URL was auto-generated. We recommend that you use the HTTP scheme.
      TODO: Change the host or pathPrefix as necessary. -->
    <data
        android:host="penging.com"
        android:pathPrefix="/activityguide"
        android:scheme="http" />
</intent-filter>
```
	
***BROWSABLE***    
若要能够从 Web 浏览器执行该 Intent，则必须将其设为 BROWSABLE。如果未设置，点击浏览器中的链接将无法解析到您的应用（在这种情况下，只有当前的移动 Web 浏览器能响应该 URL）。    
***DEFAULT***    
类别声明了您的应用可以对隐式 Intent 做出响应。    
***data标签***    

```
<data android:scheme="string"
      android:host="string"
      android:port="string"
      android:path="string"
      android:pathPattern="string"
      android:pathPrefix="string"
      android:mimeType="string" />
```
匹配：mimeType或者uri，或者mimeType＋uri    
\<scheme>://\<host>:\<port>[\<path>|\<pathPrefix>|\<pathPattern>]

*Note*    

* 必须有scheme，否则其他的属性无效
* 如果没有host，port和path属性无效

同一个intent-filter下都是一个过滤条件，如：

```
<intent-filter . . . >
    <data android:scheme="something" android:host="project.example.com" />
    . . .
</intent-filter>
```
跟如下是相同的：

```
<intent-filter . . . >
    <data android:scheme="something" />
    <data android:host="project.example.com" />
    . . .
</intent-filter>
```

**Attributes**

* android:scheme

	如果有设定data type mimeType而没有scheme，那么会假定是content:或者file:。
	scheme是大小写敏感的，默认都使用小写。

* android:host    

	host大小写敏感，默认都使用小写。

* android:port
* android:path

	匹配完整路径。

* android:pathPrefix
* android:pathPattern

	"\*" 匹配0或多次同一个字符    
	"." 匹配任意一个字符    
	".*" 匹配任意字符0次或多次    
	注意\是xml里面的特殊字符。所以，\*需要表示为\\\\\*。

* mimeType

	A MIME media type, such as image/jpeg or audio/mpeg4-generic. 也可以表示为\*/*。


### 2. Testing a Deep Link

1. After opening a project, select Run > Edit Configurations.
2. In the Run/Debug Configurations dialog, beneath Android Application, select the module you want to test.
3. Select the General tab.
4. In the Launch field, select Deep Link. 

## 二、 App Indexing
### 1. Add
右击Activity可添加。
参考网址如文章开头简介。
### 2. Test
Viewing App Indexing API Messages in the logcat Monitor.