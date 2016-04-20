title: Android-Improving-Layout-Performance
categories:
	- Android
tags:
	- Optimization
date: 2015-12-04 14:57:45
---
UI布局优化。

## 一、 Layout Hierarchies

并不是使用basic layout就一定高效率。比如嵌套Linearlayout会导致层级过深，嵌套又同时使用weight，子view会measured两次，导致额外的开销。尤其如果出现在ListView、GridView的item中。

1. Hierarchy View检查ui层次布局，看一下重复、嵌套的布局，优化ui。目标是层次越少越好。在DDMS里有Dump UI Hierarchy for UI Automator也可以看层次布局。

2. Lint工具检查，去除重复根布局，无用的子view、父view。

## 二、 include/merge/viewstub

1. \<include/>

	```
	<include layout="@layout/titlebar"/>
	```
也可以override布局中的layout_*属性，但是width和height时必须的。

	```
	<include android:id="@+id/news_title"
    	     android:layout_width="match_parent"
        	 android:layout_height="match_parent"
         	 layout="@layout/title"/>
	```
2. \<merge/> 被include的时候，merge标签将被去除。

	```
	<merge xmlns:android="http://schemas.android.com/apk/res/android">

        <Button
            android:layout_width="fill_parent"
            android:layout_height="wrap_content"
            android:text="@string/add"/>

        <Button
            android:layout_width="fill_parent"
            android:layout_height="wrap_content"
            android:text="@string/delete"/>
	</merge>
	```
3. \<viewstub> 必须指定layout属性。不占用内存。
	
	```
    <ViewStub
        android:id="@+id/stub_import"
        android:inflatedId="@+id/panel_import"
        android:layout="@layout/progress_overlay"
        android:layout_width="fill_parent"
        android:layout_height="wrap_content"
	    android:layout_gravity="bottom" />
	```
load the viewstub layout:

	```
	((ViewStub) findViewById(R.id.stub_import)).setVisibility(View.VISIBLE);
	```
    or
    
    ```
    View importPanel = ((ViewStub) findViewById(R.id.stub_import)).inflate();
	```