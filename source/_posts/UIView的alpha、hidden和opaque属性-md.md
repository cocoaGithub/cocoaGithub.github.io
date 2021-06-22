---
title: UIView的alpha、hidden和opaque属性
date: 2017-02-08 15:17:07
tags:
categories: IOS
---

# UIView的alpha、hidden和opaque属性   

## alpha   

属性值为浮点类型的值，取值范围从0到1.0，表示从完全透明到完全不透明。   
当把alpha的值设置成0后：    
<!--more-->
> 当前的UIView和其subview都会被隐藏，而不管subview的alpha值是多少  
> 当前UIView会从响应链中移除，而响应链中的下一个会成为第一响应者   
> alpha具有动画效果。当alpha为0时，跟hidden为YES时效果一样，但是alpha主要用于实现隐藏的动画效果，在动画块中将hidden设置为YES没有动画效果。   
alpha默认值是1.0

## hidden   

属性值为浮点类型，表示UIView是否隐藏，默认是NO。    
当值为YES时：  

> 当前UIView和其subview都会被隐藏   
> 当前UIView也会从响应链中移除   

## opaque     

该属性为BOOL值，UIView的默认值是YES，但UIButton等子类的默认值都是NO。  

先了解绘图系统的一些原理：屏幕上的每个像素点都是通过RGBA值（Red、Green、Blue三原色再配上Alpha透明度）表示的，当纹理（UIView在绘图系统中对应的表示项）出现重叠时，GPU会按照下面的公式计算重叠部分的像素（这就是所谓的“合成”）：

Result = Source + Destination * (1 - SourceAlpha)

Result是结果RGB值，Source为处在重叠顶部图层的RGB值，Destination为处在重叠底部图层的RGB值。  

通过公式发现：当SourceAlpha为1时，绘图系统认为下面的纹理全部被遮盖住了，Result等于Source，直接省去了计算！尤其在重叠的层数比较多的时候，完全不同考虑底下有多少层，直接用当前层的数据显示即可，这样大大节省了GPU的工作量，提高了效率。   
当opaque为YES时，SourceAlpha为1。opaque就是绘图系统向UIView开放的一个性能开关，开发者根据当前UIView的情况，将opaque设置为YES，绘图系统会根据此值进行优化。所以，如果在开发时某UIView是不透明的，就将opaque设置为YES，能优化显示效率。