title: Android-Loaders
categories:
  - Android
tags:
  - Loaders
date: 2016-03-22 17:49:02
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

1. LoaderManager    
There is only one LoaderManager per activity or fragment. But a LoaderManager can have multiple loaders.
2. LoaderManager.LoaderCallbacks
3. Loader
4. AsyncTaskLoader
5. CursorLoader

### 3. 没做深入研究，参考案例暂不收录—.—||