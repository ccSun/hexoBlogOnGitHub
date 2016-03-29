title: Android-Ui-Tools
categories:
  - Android
tags:
  - Android
  - Tools
date: 2016-01-02 11:06:05
---
有关UI操作的tool：Layout Editor，Theme Editor，Translations Editor，Vector Asset Studio，Image Asset Studio。

## 一、 Layout Editor
## 二、 Theme Editor

1. Pick a Theme
![](https://github.com/ccSun/hexoBlogOnGitHub/blob/master/source/_posts/android-ui-tools/theme_tools.png?raw=true)
2. Theme Editor
	* From an open styles.xml file, click Open editor near the top-right of the file window.
	* From the Tools menu, select Android > Theme Editor.
	
## 三、 Vector Asset Studio
Helps you ***add material icons*** and ***import Scalable Vector Graphic (SVG)*** files into your app project. SVG is an XML-based open standard of the World Wide Web Consortium (W3C). Vector Asset Studio supports the essential standard, but not all features.

####Compatibility
***Android 5.0 (API level 21)*** and higher provides vector drawable support.    
***Android 4.4 (API level 20)*** and lower doesn't support vector drawables. If your minimum API level is set at one of these API levels, Vector Asset Studio also directs Gradle to generate Portable Network Graphic(PNG) raster images of the vector drawable for backward-compatibility. 

####Benefits
Compared to raster images, vector drawables can ***reduce the size of your app** and be resized ***without loss of image quality。***
Maintaining one XML file can be easier than updating multiple raster graphics at various resolutions.

####Considerations

***The initial loading of a vector graphic can cost more CPU cycles*** than the corresponding raster image. ***Afterward, memory use and performance are similar between the two.*** We recommend that you ***limit a vector image to a maximum of 200 x 200 dp***; otherwise, it can take too long to draw.

Although vector drawables do support one or more colors, in many cases it makes sense to color icons black (android:fillColor="#FF000000"). Using this approach, you can add a tint to the vector drawable that you placed in a layout, and the icon color changes to the tint color. If the icon color isn't black, the icon color might instead blend with the tint color.

####Running Vector Android Studio
右击res文件夹，new Vector Asset.    
***Note:***    

1. Select File > Project Structure.
2. In the Project Structure dialog, select Project.
3. In the Android Plugin Version field, change the Android Plugin for Gradle version to 1.5.0 or higher.

***The default app icon size is 24 x 24 dp***, which is defined in the material design specification.

####Referring to a Vector Drawable
```
Drawable drawable = res.getDrawable(R.drawable.myimage);

if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {
   VectorDrawable vectorDrawable =  (VectorDrawable) drawable;
} else {
   BitmapDrawable bitmapDrawable = (BitmapDrawable) drawable;
}
```
## 四、 Image Asset Studio

Generates a set of icons at the appropriate resolution for each generalized screen density that your app supports.Inlcuding follow types:

* launcher icons
* action bar and tab icons
* notification icons

```
ImageView imageView = (ImageView) findViewById(R.id.myimageview);
imageView.setImageResource(R.drawable.myimage);

```
