---
title: instancetype和id区别
date: 2017-02-06 14:13:19
tags:
categories: iOS
---
# id和instancetype区别    

### 概述   
instancetype是clang3.5开始提供的一个关键字，与id一样表示未知类型的Objective-C对象。     
	
### 关联返回类型和非关联返回类型     
<!-- more -->	 
1. 关联返回类型  
	根据cocoa的命名规则，满足下述规则的方法   
	> 类方法中以alloc和new开头     
	> 实例方法中，以autorelease、init、retain、self开头    
 
	会返回一个方法所在类类型的对象，即这些方法的返回结果以方法所在的类为类型。     
	例：  
	
	```
	@interface NSObject  
	+ (id)alloc;  
	- (id)init;  
	@end
	```
   当我们使用如下方式初始化NSArray时：

	```
	NSArray *array = [[NSArray alloc] init];   
	```
   按照Cocoa的命名规则，[NSArray alloc]与[[NSArray alloc]init]返回的都为NSArray的对象。
	
2. 非关联返回类型  

   ```
	@interface NSArray    
	+ (id)constructAnArray;    
	@end
	```
	
   当我们使用如下方式初始化NSArray时：

	```
	[NSArray constructAnArray];  
	```
	
   根据Cocoa的方法命名规范，得到的返回类型就和方法声明的返回类型一样，是id。    
	
   但是如果使用instancetype作为返回类型，如下：

	```
	@interface NSArray  
	+ (instancetype)constructAnArray;  
	@end   
	```
   当使用相同方式初始化NSArray时：

	```
	[NSArray constructAnArray];  
	```
	
   得到的返回类型和方法所在类的类型相同，是NSArray*。   
	
   #### 总结：instancetype的作用就是使那些非关联返回类型的方法返回所在类的类型。      
	
## instancetype和id区别    
1. instancetype可以返回和方法所在类相同类型的对象，id只能返回未知类型的对象。
2. instancetype只能作为返回值，不能像id那样作为参数。 