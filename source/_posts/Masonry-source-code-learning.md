---
title: Masonry source code learning
date: 2017-04-18 14:33:25
tags:
categories:
---
# Masonry源码阅读   

摘要：本文介绍了在阅读Masonry库源码之后，所做的一些总结，包括该库的基本类结构、实现原理及一些常用方法等。借鉴了网上一些总结比较好的图片，根据项目开发经验，做了简单分析。

<!-- more --> 

## 一. NSLayoutConstraint约束     


```

NSLayoutConstraint *constraint = [NSLayoutConstraint constraintWithItem:(id)view1
                      attribute:(NSLayoutAttribute)attr1
                      relatedBy:(NSLayoutRelation)relation
                         toItem:(nullable id)view2
                      attribute:(NSLayoutAttribute)attr2
                     multiplier:(CGFloat)multiplier
                       constant:(CGFloat)c]
                       

```

**方法中的变量解释：**

 * view1 与 view2 ，约束关联的视图   
 * attr1 与 attr2 ：约束系统中的名词，描述视图对齐矩阵的特征，如左边、右边、中心、高度等，不存在第二项，使用 NSLayoutAttributeNotAnAttribute 占位符.   
 * relation：动词。指出属性之间如何比较    
 * mutiplier 和 constant：提供代数元素，增强约束的功能和灵活性，通过这个属性，可以指出一个视图是另一个视图的一半，也可以指定一个视图是通过另一个视图偏移某个距离得到的。即下式中m和b。   

约束本质的就是数学中的相等或者不等关系，用公式表示就是：   

```

y （关系）m * x + b    

```

**约束、层次结构与边界系统**

> 约束引用两个视图时，这两个视图必须在同一个视图层次结构，这个特别重要。  
> 对于两个涉及视图的约束，只有两种情况，一种是父视图与子视图，另一种就是视图是平级的，即它们拥有相同的祖先视图。   
> 约束通常安装在该约束所引用视图的最近父视图中。约束有必要安装在引用每个视图的公共的父视图中。约束中的数字相对于所安装视图的坐标系要有意义。   
> 视图被视为自身的父视图      


## 二、masonry主要类结构及介绍    

主要结构概览：    
   
![architecture](https://p0.meituan.net/dpnewvc/80ba460dda0970e5968353a0160bd0db154680.png)  
   
### UIView+MASAdditions

UIView的扩展，提供获取View的MASViewAttribute属性的一系列mas方法，和设置约束的三个方法：    

		- (NSArray *)mas_makeConstraints:(void(^)(MASConstraintMaker *make))block;
		
		- (NSArray *)mas_updateConstraints:(void(^)(MASConstraintMaker *make))block;
		
		- (NSArray *)mas_remakeConstraints:(void(^)(MASConstraintMaker *make))block; 
    
	
### MASConstraintMaker      

一个工厂类，主要负责创建MASConstraint对象以及把约束添加到视图上   
	  
		- (MASConstraint *)left {
		   return [self addConstraintWithLayoutAttribute:NSLayoutAttributeLeft];
		}
		- (NSArray *)install {
			 if (self.removeExisting) {
			       NSArray *installedConstraints = [MASViewConstraint installedConstraintsForView:self.view];
			       for (MASConstraint *constraint in installedConstraints) {
			           [constraint uninstall];
			       }
			   }
			   NSArray *constraints = self.constraints.copy;
			   for (MASConstraint *constraint in constraints) {
			       constraint.updateExisting = self.updateExisting;
			       [constraint install];
		      }
			   [self.constraints removeAllObjects];
			   return constraints;
			}
			
### MASViewAttribute   

MASViewAttribute 就是对 attribute 和 Item 这两个属性的封装  

	- (id)initWithView:(MAS_VIEW *)view item:(id)item layoutAttribute:(NSLayoutAttribute)layoutAttribute {
		self = [super init];
		if (!self) return nil;
		    _view = view;
		    _item = item;
		    _layoutAttribute = layoutAttribute;
		     return self;
	}      

### MASConstraint、MASViewConstraint、MASCompositeConstraint     

> MASConstraint 是个抽象类，其中有很多的方法都必须在子类中重写。MASViewConstraint 和 MASCompositeConstraint 是它的两个子类。   
		MASViewConstraint 就是对 MASViewAttribute 的封装，可以理解为一条约束对象；   
		MASCompositeConstraint 则就是约束的集合，它里面有个私有的数组用来存放多个 MASViewAttribute 对象      
	
```   
	/**
	 *	First item/view and first attribute of the NSLayoutConstraint
	 */
	@property (nonatomic, strong, readonly) MASViewAttribute *firstViewAttribute;
	
	/**
	 *	Second item/view and second attribute of the NSLayoutConstraint
	 */
	@property (nonatomic, strong, readonly) MASViewAttribute *secondViewAttribute;
	
```

 主要类详情：      
 
 ![class](https://p0.meituan.net/dpnewvc/d1404412fbadea78db87d4db454f50e8241308.png)   
 
## 三、masonry添加约束     

例：      

  ![example](https://p0.meituan.net/dpnewvc/456ad971e7226a00823b59d038b193b370841.png)
  
### 1. 在View+Additions，mas_makeConstraints方法实现如下：      

	 	- (NSArray *)mas_makeConstraints:(void(^)(MASConstraintMaker *))block {
	    	self.translatesAutoresizingMaskIntoConstraints = NO;
	    	MASConstraintMaker *constraintMaker = [[MASConstraintMaker alloc] initWithView:self];
	   	 	block(constraintMaker);
	    	return [constraintMaker install];
	    }
  
 	 
> 首先把 translatesAutoresizingMaskIntoConstraints 属性设置为 NO   
 	
 translatesAutoresizingMaskIntoConstraints默认为YES，也就是按照默认的autoresizingMask进行计算；设置为NO之后，则可以使用更灵活的Autolayout（或者Masonry）之类的工具进行自动布局     

> 初始化 MASConstraintMaker 工厂实例对象并保存了当前视图 self.view   
把初始化好的 MASConstraintMaker 对象传入 block，回调给外面配置约束属性。    
最后调用 install 方法，把配置好的约束添加到视图上去。     

### 2. block调用后，MASConstaintMaker中的实现    
 
	**make.left**   
	    
  调用链如下：    

		- (MASConstraint *)left {
	    	return [self addConstraintWithLayoutAttribute:NSLayoutAttributeLeft];
		}
	
		- (MASConstraint *)addConstraintWithLayoutAttribute:(NSLayoutAttribute)layoutAttribute {
	    	return [self constraint:nil addConstraintWithLayoutAttribute:layoutAttribute];
		}

		- (MASConstraint *)constraint:(MASConstraint *)constraint addConstraintWithLayoutAttribute:(NSLayoutAttribute)layoutAttribute {
		    MASViewAttribute *viewAttribute = [[MASViewAttribute alloc] initWithView:self.view layoutAttribute:layoutAttribute];
		    MASViewConstraint *newConstraint = [[MASViewConstraint alloc] initWithFirstViewAttribute:viewAttribute];
		    if ([constraint isKindOfClass:MASViewConstraint.class]) {
		        //replace with composite constraint
		        NSArray *children = @[constraint, newConstraint];
		        MASCompositeConstraint *compositeConstraint = [[MASCompositeConstraint alloc] initWithChildren:children];
		        compositeConstraint.delegate = self;
		        [self constraint:constraint shouldBeReplacedWithConstraint:compositeConstraint];
		        return compositeConstraint;
		    }
		    if (!constraint) {
		        newConstraint.delegate = self;
		        [self.constraints addObject:newConstraint];
		    }
		    return newConstraint;
		 }
     
由代码可知，返回一个MASViewConstraint对象，主要执行一下步骤：   
	
> 初始化了 MASViewAttribute 对象并保存了 view、item以及NSLayoutAttribute三个属性。   
	
```
	- (id)initWithView:(MAS_VIEW *)view layoutAttribute:(NSLayoutAttribute)layoutAttribute {
	    self = [self initWithView:view item:view layoutAttribute:layoutAttribute];
	    return self;
	}
	
	- (id)initWithView:(MAS_VIEW *)view item:(id)item layoutAttribute:(NSLayoutAttribute)layoutAttribute {
	    self = [super init];
	    if (!self) return nil;
	
	    _view = view;
	    _item = item;
	    _layoutAttribute = layoutAttribute;
	
	    return self;
	}

```

> 然后初始化了 MASViewConstraint 对象，内部配置了些默认参数并保存了第一个约束参数 MASViewAttribute。   
	
	- (id)initWithFirstViewAttribute:(MASViewAttribute *)firstViewAttribute {
	    self = [super init];
	    if (!self) return nil;
	
	    _firstViewAttribute = firstViewAttribute;
	    self.layoutPriority = MASLayoutPriorityRequired;
	    self.layoutMultiplier = 1;
	
	    return self;
	}
	
> 最后设置 MASViewConstraint对象代理并添加到一开始准备好的 self.constraints 数组中。
	
**make.left.equalTo(@30)**   
	在 make.height 返回 MASViewConstraint 对象后，会继续在这个链式的语法中调用下一个方法来指定约束的关系       

	- (MASConstraint * (^)(id))equalTo {
	    return ^id(id attribute) {
	        return self.equalToWithRelation(attribute, NSLayoutRelationEqual);
	    };
	}
	
	
	- (MASConstraint * (^)(id))greaterThanOrEqualTo {
	    return ^id(id attribute) {
	        return self.equalToWithRelation(attribute, NSLayoutRelationGreaterThanOrEqual);
	    };
	}
	

	- (MASConstraint * (^)(id))lessThanOrEqualTo {
	    return ^id(id attribute) {
	        return self.equalToWithRelation(attribute, NSLayoutRelationLessThanOrEqual);
	    };
	}
   
	
  MASConstraint 是个抽象类，具体实现都在它的两个子类中，equalTo(self) 的调用链如下：   
   
	  //MASViewConstaint 
	  - (MASConstraint * (^)(id, NSLayoutRelation))equalToWithRelation {   
	    return ^id(id attribute, NSLayoutRelation relation) {
	        if ([attribute isKindOfClass:NSArray.class]) {
	            ......
	        } else {
	            NSAssert(!self.hasLayoutRelation || self.layoutRelation == relation && [attribute isKindOfClass:NSValue.class], @"Redefinition of constraint relation");
	            self.layoutRelation = relation;
	            self.secondViewAttribute = attribute;
	            return self;
	        }
	    };
	 	}
	
		//MASCompositeConstraint
		- (MASConstraint * (^)(id, NSLayoutRelation))equalToWithRelation {
	    return ^id(id attr, NSLayoutRelation relation) {
	        for (MASConstraint *constraint in self.childConstraints.copy) {
	            constraint.equalToWithRelation(attr, relation);
	        }
	        return self;
	    };
		}
 
   
   > 首先是 self.layoutRelation 保存了约束关系且重写了 set 方法，在里面用 self.hasLayoutRelation 这个 BOOL 标识已经有约束关系   
   
```  
   //MASViewConstaint 
   - (void)setLayoutRelation:(NSLayoutRelation)layoutRelation {
    	_layoutRelation = layoutRelation;
    	self.hasLayoutRelation = YES;
	}
	
```
   > 然后同样是重写了 self.secondViewAttribute 的 set 方法，这里会根据不同的情况做不同的操作   
   
   
![equal](https://p1.meituan.net/dpnewvc/0958c0f564a87d0f0935f8f0accd8993199335.png)       
	
### 3. install constraint    
调用链如下：      

		//MASConstaintMaker   
		- (NSArray *)install {
	    	if (self.removeExisting) {
	       	 NSArray *installedConstraints = [MASViewConstraint installedConstraintsForView:self.view];
	        	for (MASConstraint *constraint in installedConstraints) {
	          	  [constraint uninstall];
	       	 }
	   		}
	    	NSArray *constraints = self.constraints.copy;
	    	for (MASConstraint *constraint in constraints) {
	     	   constraint.updateExisting = self.updateExisting;
	     	   [constraint install];
	    	}
	    	[self.constraints removeAllObjects];
	    	return constraints;
		}     
		
		//MASViewConstraint
		
		- (void)install {
		  	 MAS_VIEW *firstLayoutItem = self.firstViewAttribute.item;
		    NSLayoutAttribute firstLayoutAttribute = self.firstViewAttribute.layoutAttribute;
		    MAS_VIEW *secondLayoutItem = self.secondViewAttribute.item;
		    NSLayoutAttribute secondLayoutAttribute = self.secondViewAttribute.layoutAttribute;
		
		    // eg make.left.equalTo(@10)
		    if (!self.firstViewAttribute.isSizeAttribute && !self.secondViewAttribute) {
		        secondLayoutItem = self.firstViewAttribute.view.superview;
		        secondLayoutAttribute = firstLayoutAttribute;
		    }
		    
		    MASLayoutConstraint *layoutConstraint
		        = [MASLayoutConstraint constraintWithItem:firstLayoutItem
		                                        attribute:firstLayoutAttribute
		                                        relatedBy:self.layoutRelation
		                                           toItem:secondLayoutItem
		                                        attribute:secondLayoutAttribute
		                                       multiplier:self.layoutMultiplier
		                                         constant:self.layoutConstant];
		    
		    if (self.secondViewAttribute.view) {
		        MAS_VIEW *closestCommonSuperview = [self.firstViewAttribute.view mas_closestCommonSuperview:self.secondViewAttribute.view];
		        self.installedView = closestCommonSuperview;
		    } else if (self.firstViewAttribute.isSizeAttribute) {
		        self.installedView = self.firstViewAttribute.view;
		    } else {
		        self.installedView = self.firstViewAttribute.view.superview;
		    }
		
		    MASLayoutConstraint *existingConstraint = nil;
		    if (self.updateExisting) {
		    }
		    if (existingConstraint) {    
		    } else {    
		    }
		 }
  
	
 ![constraint_similar](https://p1.meituan.net/dpnewvc/a669d9ad193a639909816b9feeb036fb231114.png)
	
	
	
## 四、masonry的链式调用的实现（工厂方法）

**make.left.top.equalTo(self.view).offset(30)**    
主要通过 MASConstraintDelegate实现。    
  
	@protocol MASConstraintDelegate <NSObject>
	
	- (void)constraint:(MASConstraint *)constraint shouldBeReplacedWithConstraint:(MASConstraint *)replacementConstraint;
	
	- (MASConstraint *)constraint:(MASConstraint *)constraint addConstraintWithLayoutAttribute:(NSLayoutAttribute)layoutAttribute;
	
	@end
    
 
### 1. MASConstraintMaker 实现该delegate      

 ![make_create_constraint](https://p1.meituan.net/dpnewvc/cb71d5e222edcc90016ac69e3c3ab987363184.png)      
 
 方法中生成的MASViewConstraint或MASCompositeConstraint对象设置delegate为self     
 
### 2. MASViewConstraint或MASCompositeConstraint中，通过delegate调用工厂方法
MASViewConstraint或MASCompositeConstraint中，通过delegate调用 MASConstraintMaker 工厂类中的 constraint:addConstraintWithLayoutAttribute: 方法生成新的MASConstraint对象   
     
		//MASConstraint 
		 - (MASConstraint *)right {
	    	return [self addConstraintWithLayoutAttribute:NSLayoutAttributeRight];
		}
	
		//MASViewConstraint
		- (MASConstraint *)addConstraintWithLayoutAttribute:(NSLayoutAttribute)layoutAttribute {
		    NSAssert(!self.hasLayoutRelation, @"Attributes should be chained before defining the constraint relation");
		
		    return [self.delegate constraint:self addConstraintWithLayoutAttribute:layoutAttribute];
		}
		
		//MASCompositeConstraint
		- (MASConstraint *)addConstraintWithLayoutAttribute:(NSLayoutAttribute)layoutAttribute {
	    	[self constraint:self addConstraintWithLayoutAttribute:layoutAttribute];
	    	return self;
	    }
	    - (MASConstraint *)constraint:(MASConstraint __unused *)constraint addConstraintWithLayoutAttribute:(NSLayoutAttribute)layoutAttribute {
	    	id<MASConstraintDelegate> strongDelegate = self.delegate;
	    	MASConstraint *newConstraint = [strongDelegate constraint:self addConstraintWithLayoutAttribute:layoutAttribute];
	    	newConstraint.delegate = self;
	    	[self.childConstraints addObject:newConstraint];
	    	return newConstraint;
		}
  
### 3. MASConstraint中实例方法equalTo

	```
	- (MASConstraint * (^)(id))equalTo {
	    return ^id(id attribute) {
	        return self.equalToWithRelation(attribute, NSLayoutRelationEqual);
	    };
	}
	```
这个方法，没有参数，返回值是一个类型为（MASConstraint * （^）(id)），即返回值是MASConstraint对象，接受一个id类型参数的block闭包。 


## 五、使用    

1. \#define MAS_SHORTHAND_GLOBALS使用全局宏定义，可以使equalTo等效于mas_equalTo
2. \#define MAS_SHORTHAND使用全局宏定义, 可以在调用masonry方法的时候不使用mas\_前缀    
3. view数组设置约束     
	例：  
	  
	```    
	NSMutableArray *btnArray = [NSMutableArray arrayWithCapacity:3];
    for (int i=0; i< 3; i++) {
        UIButton *btn1 = [[UIButton alloc] init];
        [btn1 setBackgroundColor:[UIColor redColor]];
        [btn1 setTitle:[NSString stringWithFormat:@"btn%d",i] forState:UIControlStateNormal];
        [self.view addSubview:btn1];
        [btnArray addObject:btn1];
    }
    [btnArray mas_distributeViewsAlongAxis:MASAxisTypeHorizontal withFixedSpacing:30 leadSpacing:15 tailSpacing:15];
    [btnArray mas_makeConstraints:^(MASConstraintMaker *make) {
        make.top.equalTo(label2.bottom).offset(30);
        make.height.equalTo(@30);
    }];
	```
	NSArray中新增的两个方法： 2个或2个以上的控件等间隔排序   

		//  distribute with fixed spacing
		- (void)mas_distributeViewsAlongAxis:(MASAxisType)axisType withFixedSpacing:(CGFloat)fixedSpacing leadSpacing:(CGFloat)leadSpacing tailSpacing:(CGFloat)tailSpacing;
		
		//  distribute with fixed item size
		- (void)mas_distributeViewsAlongAxis:(MASAxisType)axisType withFixedItemLength:(CGFloat)fixedItemLength leadSpacing:(CGFloat)leadSpacing tailSpacing:(CGFloat)tailSpacing; 
    
4. multipliedBy、dividedBy

		make.width.equalTo(self.view).multipliedBy(0.5);
		make.width.equalTo(self.view).dividedBy(3);
	    
5. 约束的优先级    

	> .priority允许你指定一个精确的优先级,数值越大优先级越高.最高1000   
.priorityHigh等价于 UILayoutPriorityDefaultHigh .优先级值为 750   
.priorityMedium介于高优先级和低优先级之间,优先级值在 250~750之间  
.priorityLow等价于 UILayoutPriorityDefaultLow , 优先级值为 250    

6. 与父视图属性相等    
    
		make.left.right.equalTo(superView);
		
		make.left.right.equalTo(@0);   
		
		make.left.right.mas_equalTo(0);  
	
7. equalTo返回的block的参数    

		make.left.equalTo(self.view);
		
		make.left.equalTo(self.view.mas_left);
		
		make.left.equalTo(@15);
		
		make.left.equalTo(viewArray);

8. 动态修改约束    

	```   
	 [input mas_makeConstraints:^(MASConstraintMaker *make) {
        self.constraint = make.left.equalTo(self.view).offset(15);
        ......
    }];
    
    - (void)changeConstraint {
    self.constraint.offset = 100;
    [self.view layoutIfNeeded];
	}

	```

## 六、总结  

#### AutoLayout关于更新的几个方法的区别

> setNeedsLayout：告知页面需要更新，但是不会立刻开始更新。执行后会立刻调用layoutSubviews。   
	layoutIfNeeded：告知页面布局立刻更新。所以一般都会和setNeedsLayout一起使用。如果希望立刻生成新的frame需要调用此方法，利用这点一般布局动画可以在更新布局后直接使用这个方法让动画生效。    
	layoutSubviews：系统重写布局    
	setNeedsUpdateConstraints：告知需要更新约束，但是不会立刻开始    
	updateConstraintsIfNeeded：告知立刻更新约束    
	updateConstraints：系统更新约束     
	
	  
源码中没有用[]调用left、equalto等方法，而是用的点语法来调用，是因为OC中虽然推荐只有对象属性的get、set方法使用点语法，其实一般的示例方法也是可以用点语法这样做的。但不推荐大家这样做。         





