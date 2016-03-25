title: Android-Code-Tools
categories:
  - Android
tags:
  - Android
  - Lint
  - Annotations
  - DeepLink
  - AppIndex
  - Optimization
date: 2016-01-25 17:20:08
---
Lint检查潜在错误、优化；Annotations优化参数定义，增加可读性；DeepLink实现Android和H5页面跳转；App Index可以网页搜索app内部的功能。Android和iOS都可添加deep link以及app index：https://developers.google.com/app-indexing/introduction。

## 一、 Lint
The Lint tool is a static code analysis tool that checks your Android project source files for potential bugs and optimization improvements.

启动： Analyze -> Inspect codes.    
配置： File -> Settings -> Project Settings

***检查的内容：***

1. XML resource files contain unused namespaces
2. 使用的deprecated elements or API

## 二、 Annotations

Lint不能检查到的的正确性，比如xml中使用string的地方，引入了一个color。因为都是int型。
Add dependency import android.support.annotation.   
Annotations allow you to provide hints to code inspections tools like lint, to help detect these, more subtle code problems. Annotations可以帮助Lint检查更多错误。

### 1. Adding Resource Annotations
1. ***@StringRes***    
References a R.string resource.
2. ***@DrawableRes***    
References a Drawable resource.
3. ***@ColorRes***    
References a Color resource.
4. ***@InterpolatorRes***    
References a Interpolator resource.
5. ***@AnyRes***    
References any type of R. resource.

```
import android.support.annotation.StringRes;
...
    public abstract void setTitle(@StringRes int resId);
    ...
```

### 2. Adding Nullness Annotations

Add ***@Nullable*** and ***@NonNull*** annotations to check the nullness of a given variable, parameter, or return value.


```
import android.support.annotation.NonNull;
...

    /** Add support for inflating the <fragment> tag. */
    @NonNull
    @Override
    public View onCreateView(String name, @NonNull Context context,
      @NonNull AttributeSet attrs) {
      ...
      }
...
```

### 3. Adding Thread Annotations

1. ***@UiThread***
2. ***@MainThread***
3. ***@WorkerThread***
4. ***@BinderThread***

### 4. Adding Value Constraint Annotations

1. ***@IntRange***
2. ***@FloatRange***
3. ***@Size***    
checks the size of a collection or array, as well as the length of a string

```
public void setAlpha(@IntRange(from=0,to=255) int alpha) { … }
...
int[] location = new int[3];
button.getLocationOnScreen(@Size(min=1) location);
// @Size(2) annotation to validate that an array contains exactly two values

```

### 4. Adding Permission Annotations

1. ***@RequiresPermission***

	```
	@RequiresPermission(Manifest.permission.SET_WALLPAPER)
	public abstract void setWallpaper(Bitmap bitmap) throws IOException;
	```
2. ***allOf***

	```
	@RequiresPermission(allOf = {
    	Manifest.permission.READ_EXTERNAL_STORAGE,
	    Manifest.permission.WRITE_EXTERNAL_STORAGE})
	public static final void copyFile(String dest, String source) {
    	...
	}
	```
### 5. Adding CheckResults Annotations
确保返回值被使用。但是语法没搞明白。

### 6. Adding CallSuper Annotations
```
@CallSuper
protected void onCreate(Bundle savedInstanceState) {
}
```

### 7. Creating Enumerated Annotations
我后来能用到这个语法吗。。呵呵呵呵

## 三、 DeepLink

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
data标签必须有scheme属性；
android:path 属性或者其变体（pathPattern 或 pathPrefix）来区分对于不同的 URI 路径。

### 2. Testing a Deep Link

1. After opening a project, select Run > Edit Configurations.
2. In the Run/Debug Configurations dialog, beneath Android Application, select the module you want to test.
3. Select the General tab.
4. In the Launch field, select Deep Link. 

## 四、 App Indexing
### 1. Add
右击Activity可添加。
参考网址如文章开头简介。
### 2. Test
Viewing App Indexing API Messages in the logcat Monitor.