---
title: OC协议中声明属性
date: 2021-06-23 15:55:03
tags: 类
categories: IOS
---

### 一、@protocol中能不能声明属性

#### iOS的UIKit中就有协议中声明属性的例子：

打开UIKit中的协议UIApplicationDelegate声明就可以看到属性window；

``` C
@protocol UIApplicationDelegate<NSObject>

@optional

- (void)applicationDidFinishLaunching:(UIApplication *)application;
- (BOOL)application:(UIApplication *)application willFinishLaunchingWithOptions:(nullable NSDictionary<UIApplicationLaunchOptionsKey, id> *)launchOptions API_AVAILABLE(ios(6.0));
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(nullable NSDictionary<UIApplicationLaunchOptionsKey, id> *)launchOptions API_AVAILABLE(ios(3.0));

@property (nullable, nonatomic, strong) UIWindow *window API_AVAILABLE(ios(5.0));

@end
```

> 结论：OC语言的协议里面是支持声明属性的

<!-- more -->

### 二、协议中的属性含义

``` C
// 协议
@protocol CCTestCellVoDelegate  <NSObject>
@required
@property(nonatomic, assign) BOOL borderShow;
@end

// 实现类
@interface CCTestCellDTO : NSObject <CCTestCellVoDelegate>
@end
@implementation CCTestCellDTO
  
@end
```

> **未实现警告**  
当我们在协议中定义一个必须实现(@ required修饰)的属性以后，如果实现类没有对这个属性做任何实现那么XCode中实现类中就会发出警告

实现类中添加如下方法，警告⚠️消失

``` C
@implementation CCTestCellDTO

- (void)setBorderShow:(BOOL)borderShow {
}

- (BOOL)borderShow {
    return 0;
}
@end
```

> **总结：**  
在协议中声明属性其实和在其中定义方法一样只是声明了getter和setter方法，并没有具体实现，所以当这个协议属性修饰符为@ required时，如果不实现编译器就会报出警告

### 三、协议中的属性如何实现

#### 方式一：在.m实现类中添加@synthesize xxx 通知编译器自动合成setter和getter方法，并生成xxx格式成员变量

```C
// 协议
@protocol CCTestCellVoDelegate  <NSObject>
@required
@property(nonatomic, assign) BOOL borderShow;
@end

// 实现类
@interface CCTestCellDTO : NSObject <CCTestCellVoDelegate>
@end
@implementation CCTestCellDTO
// 自动合成getter/setter方法，并声明成员变量 xxx
@synthesize borderShow; 
// 也可以指定成员变量名 _xxx
// @synthesize borderShow = _borderShow; 

@end
```

关键字 **@synthesize** 会默认去访问 **xxx** 的同名变量，如果找不到则会自动生成一个；@synthesize也可以指定成员变量名。

#### 方式二、在.m实现文件中添加合成 xxx 属性的成员变量 _xxx 和对应的getter和setter方法

```C
// 协议
@protocol CCTestCellVoDelegate  <NSObject>
@required
@property(nonatomic, assign) BOOL borderShow;
@end

// 实现类
@interface CCTestCellDTO : NSObject <CCTestCellVoDelegate>
@end
@implementation CCTestCellDTO {
    BOOL _borderShow;
}

- (void)setBorderShow:(BOOL)borderShow {
    _borderShow = borderShow;
}
- (BOOL)borderShow {
    return _borderShow;
}
@end
```

#### 方式三、在实现类中通过@property重复声明 xxx 属性变量，由于属性变量编译器自动合成_xxx成员变量和对应的getter/setter方法，则也同时实现了协议中属性的setter/getter方法

```C
// 协议
@protocol CCTestCellVoDelegate  <NSObject>
@required
@property(nonatomic, assign) BOOL borderShow;
@end

// 实现类
@interface CCTestCellDTO : NSObject <CCTestCellVoDelegate>
@property(nonatomic, assign) BOOL borderShow;
@end
@implementation CCTestCellDTO
@end
```

### 四、总结

> **结论：**  
OC语言的协议里面是支持声明属性的，而在协议中声明属性其实和在其中定义方法一样只是声明了getter和setter方法，并没有具体实现，所以当这个协议属性修饰符为@ required时，如果不实现编译器就会报出警告，最简单的方式就是加上属性同步语句@synthesize propertyName;

**附：**

关于实现协议属性的几种不同方式，实现类成员变量情况：

```C
// 协议
@protocol CCTestCellVoDelegate  <NSObject>
@optional
@property(nonatomic, assign) NSInteger position;
@property(nonatomic, assign) CGFloat cellMarginLeft;
@property(nonatomic, assign) CGFloat cellPaddingLeft;
@end

// 实现类
@interface CCTestCellDTO 
@property(nonatomic, assign) CCTestCellCardType cardType;
@end
@implementation CCTestCellDTO 
@end

#pragma mark - 测试不同实现方式，实现类的成员变量情况
// scene 1
@implementation CCTestCellDTO {
    CGFloat _cellMarginLeft;
}
@synthesize position;
@synthesize cellMarginLeft;
@synthesize cellPaddingLeft = _cellPaddingLeft;
@end
/*
Ivar:_cellMarginLeft
Ivar:position
Ivar:cellMarginLeft
Ivar:_cellPaddingLeft

Ivar:_cardType
```

> 实现类自己@property声明的属性，生成_propertyName 形式变量

> @synthesize声明的属性，默认生成propertyName同名变量，也可以指定变量名

参考链接：
https://www.jianshu.com/p/fdbee61fedeb