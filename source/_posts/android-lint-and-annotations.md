title: Android-Code-Tools
categories:
  - Android
tags:
  - Lint
  - Annotations
  - Optimization
  - Tools
date: 2015-07-30 17:20:08
---
Lint检查潜在错误、优化；Annotations优化参数定义，增加可读性。

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
