---
title: IOS view的frame和bounds区别
date: 2017-01-23 19:12:15
tags:  
categories: iOS
---
      

先看一下代码：     

	-(CGRect)frame{
	    return CGRectMake(self.frame.origin.x,self.frame.origin.y,self.frame.size.width,self.frame.size.height);
	}
	-(CGRect)bounds{
	    return CGRectMake(0,0,self.frame.size.width,self.frame.size.height);
	}

bounds的原点是(0,0)点，就是View本身的坐标系统，默认都是0，0点，除非人为setbounds；     
frame的原点是任意的，相对于父视图中的坐标位置。        

<!-- more -->

参考下图示意：    
![](/img/frame&bounds.jpg)      

1. frame:该View在父View坐标系统中的位置和大小，参照点是父View的坐标系统；     
2. bounds：该View在本地坐标系统中的位置和大小，参照点是本地坐标系统，即View自己的坐标系统，默认以0，0为起点；     
3. center：该View的中心点在父View坐标系统中的位置，参照点是父View坐标系统。      
4. 通过修改View的bounds属性可以修改本地坐标系统的原点位置。 例： 
	  
	  	[view setBounds:CGRectMake(-20, -20, 300, 300)];    
	  	
	  则View坐标系的原点为（-20，-20）   
	  bounds参考自己坐标系，可以修改自己坐标系的原点位置，进而影响到“子view”的显示位置
   

demo演示：       

	UIView *view1 = [[UIView alloc] initWithFrame:CGRectMake(20, 20, 280, 250)];  
	[view1 setBounds:CGRectMake(-20, -20, 280, 250)];  
	view1.backgroundColor = [UIColor redColor];  
	[self.view addSubview:view1];//添加到self.view  
	NSLog(@"view1 frame:%@========view1 bounds:%@",NSStringFromCGRect(view1.frame),NSStringFromCGRect(view1.bounds));  
	  
	UIView *view2 = [[UIView alloc] initWithFrame:CGRectMake(0, 0, 100, 100)];  
	view2.backgroundColor = [UIColor yellowColor];  
	[view1 addSubview:view2];//添加到view1上,[此时view1坐标系左上角起点为(-20,-20)]  
	NSLog(@"view2 frame:%@========view2 bounds:%@",NSStringFromCGRect(view2.frame),NSStringFromCGRect(view2.bounds));     
	 
结果如图所示：   
![](/img/frame&boundsDemo.jpg)     

为何（-20，-20）的偏移量，却可以让view2向右下角移动呢？   
这是因为setBounds的作用是：强制将自己（view1）坐标系的左上角点，改为（-20，-20）。那么view1的原点（0，0），自然就向在右下方偏移（20，20）。

