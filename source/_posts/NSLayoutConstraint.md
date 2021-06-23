---
title: NSLayoutConstraint
date: 2017-01-23 14:48:54
tags: 布局
categories: IOS
---
## NSLayoutConstraint     

Notes:在使用Auto Layout时，首先需要将视图的setTranslatesAutoresizingMaskIntoConstraints属性设置为NO。这个属性默认为YES，当它为 YES 时，运行时系统会自动将 Autoresizing Mask 转换为 Auto Layout 的约束，这些约束很有可能会和我们自己添加的产生冲突。       

Auto Layout中约束对应的类为NSLayoutConstraint，一个NSLayoutConstraint实例代表一条约束。      
<!-- more -->       
主要介绍constraintWithItem：    

	[NSLayoutConstraint constraintWithItem:(id)item
	                             attribute:(NSLayoutAttribute)attribute
	                             relatedBy:(NSLayoutRelation)relation
	                                toItem:(id)otherItem
	                             attribute:(NSLayoutAttribute)otherAttribute
	                            multiplier:(CGFloat)multiplier
	                              constant:(CGFloat)constant]   
	                              
参数说明：      
item：指定约束左边的视图view1     
attribute：指定view1的属性attr1   
relation：指定左右两边的视图的关系      
otherItem：指定约束右边的视图     
otherAttribute：指定View2的属性attr2    
multiplier：指定一个与view2属性相乘的乘数multiplier    
constant：指定一个与View2属性相加的浮点数      
函数的对照公式如下：     
View1.attr1  <relation>  view2.attr2 * multiplier + constant     

Notes:若约束里不需要第二个View，则要将第四个参数设为nil，第五个参数设为NSLayoutAttributeNotAnAttribute。     

tip：视图的属性和关系的值      

	typedef NS_ENUM(NSInteger, NSLayoutRelation) {
	    NSLayoutRelationLessThanOrEqual = -1,          //小于等于
	    NSLayoutRelationEqual = 0,                     //等于
	    NSLayoutRelationGreaterThanOrEqual = 1,        //大于等于
	};
	typedef NS_ENUM(NSInteger, NSLayoutAttribute) {
	    NSLayoutAttributeLeft = 1,                     //左侧
	    NSLayoutAttributeRight,                        //右侧
	    NSLayoutAttributeTop,                          //上方
	    NSLayoutAttributeBottom,                       //下方
	    NSLayoutAttributeLeading,                      //首部
	    NSLayoutAttributeTrailing,                     //尾部
	    NSLayoutAttributeWidth,                        //宽度
	    NSLayoutAttributeHeight,                       //高度
	    NSLayoutAttributeCenterX,                      //X轴中心
	    NSLayoutAttributeCenterY,                      //Y轴中心
	    NSLayoutAttributeBaseline,                     //文本底标线
	                                                                                                                                                    
	    NSLayoutAttributeNotAnAttribute = 0            //没有属性
	};      
	
例：    

	[NSLayoutConstraint constraintWithItem:view1
	                             attribute:NSLayoutAttributeLeft
	                             relatedBy:NSLayoutRelationEqual
	                                toItem:view2
	                             attribute:NSLayoutAttributeRight
	                            multiplier:1
	                              constant:10]       
	                              
翻译过来就是：view1的左侧距离view2的右侧10个点的位置。