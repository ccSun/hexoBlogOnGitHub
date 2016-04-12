title: Android-Icon-Size
categories:
  - Android
tags:
  - IconSize
date: 2015-06-04 16:05:33
---

Launcher Icons存放到mipmap目录。但是官方建议也就只有launcher icons放到mipmap目录。Android使用mipmap目录下的资源，在scale上会有更好的性能优化。

App资源优化会移除一些未使用的屏幕密度资源，在某些屏上会模糊。

Apps should ***use the mipmap/ resource folders for launcher icons***. The Android system preserves these resources regardless of density stripping, and ensures that launcher apps can pick icons with the best resolution for display.     

Moving all densities of your launcher icons to density-specific res/mipmap/ folders (for example res/mipmap-mdpi/ and res/mipmap-xxxhdpi/)。

***Note:***添加一个当前更高的icon以增加对更高分辨率的兼容性。
    
***icon Size:***    

| name | DPI | iconSize | 四边各留像素 | 实际图像size |
|-|-|-|-|-|
| ldpi | 120 | 36x36 | 1px | 34x34 |
| mdpi | 160 | 48x48 | 1px | 46x46 |
| hdpi | 240 | 72x72 | 2px | 68x68 |
| xhdpi| 320 | 96x96 | 4px | 88x88 |
| xxhdpi| 480 | 144x144 | 8px | 128x128 |
| xxxhdpi| 640 | 192x192| 12px | 168x168 |