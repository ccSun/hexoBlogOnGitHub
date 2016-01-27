title: Object-c Diary
categories:
  - iOS
tags:
  - iOS
  - Object-c
date: 2016-01-26 17:08:25
---
Object-c的基础知识点整理

## 1. Object-c

1. OC里NS前缀的来生
Cocoa 是从1980年代由 NeXT 开发的编程环境 NeXTSTEP 和 OPENSTEP 演变而来，这点可由其类别之名皆以 NS 前缀（代表NeXTSTEP）看出端倪。苹果电脑公司在1996年12月收购了NeXT。  

2. CF前缀
Foundation框架是以CoreFoundation为基础创建的。CoreFoundation是纯C写的。变量名以CF开头。  

3. CG前缀
CG前缀是Core Graphics框架提供，用于2D渲染。如CGPoint，CGSize。  

4. 所有的Object－c对象都是动态分配的，动态分配会损耗时间。所以Foundation中有一些数据类型是用C写的。  

5. 定义函数参数 ... 表示可以传入多个以逗号间隔的参数  

6. 定义函数前缀 ＋ ，表示当前方法属于类，而不是实例对象。通常用于工厂方法创建对象实例，也用于全局数据。 

7. OC创建一个对象，包含指向 超类，类名，类方法列表的指针，和一个long数据表示实例对象的大小（以字节为单位）。


## 2. Foundation Kit
###(1). 简介

1. Cocoa常用的是Foundation和Application Kit。包含所有的用户界面UI和高级类。

###(2). 常用数据类型

* 范围
cool在"Objective-C is a cool language"中的区间[17,4]    
(1)NSRange range = {17, 4};    
(2)NSRange range = NSMakeRange(17,4);     

```
	typedef struct _NSRANGE{
		unsigned int location;
		unsigned int length;
	} NSRange;
```
* 几何数据

```
	CGPoint 	点坐标
	CGSize		长度宽度
	CGRect		矩形
```

* 字符串

长度

```
	NSString *height = [NSString stringWithFormat:@"lenght:%d", 5];	
	NSUInteger len = [height length]; // length返回准确长度，适用中文，strlen只计算字节数。	
```
大小

```
	if(str1 compare: str2 options: NSCaseInsensitiveSearch)
		[@"100" compare: @"99"]   NSOrderedAscending
		[@"100" compare: @"99" options:NSNumericSearch]  NSOrderedDescending        
```

包含

```
	[filename hasPrefix:@"draft"]
	[filename hasSuffix:@".rmvb"]
	NSRange range = [filename rangeOfString:@chapter"]
```

可变长度

```
	NSMutableString *string = [NSMutableString stringWithCapacity:42]; // 给一个大约的范围
	[string appendString: @"hello tom god"];
	[string appendFormat: @"human %d", 39];
	Range range = [string rangeOfString:@"tom"];
	range.length++; // eat the space that follows
	[string deleteCharactersInRange:range]
```

* 颜色

```
	NSColor *blue = [NSColor blueColor];
	UIColor *blue = [UIColor blueColor];
```

＊几何

```
	// NSArray 只能存OC的对象，不能存C原始类型 int float enum struct
	NSArray *array = [NSArray arrayWithObjects:@"one",@"two",@"three",nil];
	NSArray *array = @[@"one", @"two", @"three"];
```