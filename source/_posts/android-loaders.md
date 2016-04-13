title: Android-Loaders
categories:
  - Android
tags:
  - Loaders
date: 2015-11-23 17:49:02
---
常用的是Android异步加载数据的CursorLoader, 继承自AsyncTaskLoader。可以实现MVVM的数据监测。

## 一、 Loaders Introduction

### 1. Brief Introduction

1. They are available to every Activity and Fragment.
2. They provide asynchronous loading of data.
3. They monitor the source of their data and deliver new results when the content changes.
4. They automatically reconnect to the last loader's cursor when being recreated after a configuration change. Thus, they don't need to re-query their data.

使用场合：通讯录动态查询，ListView的数据动态发生变化更新UI。


### 2. Loader API Summary

1. ***LoaderManager***     
An abstract class. There is only one LoaderManager per activity or fragment. But a LoaderManager can have multiple loaders.
2. ***LoaderManager.LoaderCallbacks***

	```
	public interface LoaderCallbacks<D> {
	
    	Loader<D> onCreateLoader(int var1, Bundle var2);

	    void onLoadFinished(Loader<D> var1, D var2);

	    void onLoaderReset(Loader<D> var1);
	}
	```
3. ***Loader***    
An abstract class that performs asynchronous loading of data. This is the base class for a loader. You would typically use CursorLoader, but you can implement your own subclass. 
4. ***AsyncTaskLoader***    
Abstract loader that provides an AsyncTask to do the work.
5. ***CursorLoader***
A subclass of AsyncTaskLoader that queries the ContentResolver and returns a Cursor. Using this loader is the best way to asynchronously load data from a ContentProvider, instead of performing a managed query through the fragment or activity's APIs.


### 3. 没做深入研究~

http://developer.android.com/guide/components/loaders.html#app