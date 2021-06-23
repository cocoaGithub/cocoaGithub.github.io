---
title: iOS 项目引入YogaKit
date: 2019-01-05 16:30:25
tags: iOS flex布局
categories: IOS
---

[Flexbox基本概念](#header1)
[Flexbox解决了什么？](#header2)
[iOS基于flexbox的布局](#header3)
[YogaKit](#header4)
[Yoga : 使用跨平台布局引擎](#header41
[使用](#header42)
[接入和使用中的问题](#header43)

<!-- more -->

<h3 id="header1">Flexbox基本概念</h3>

FlexBox 是一种 UI 布局方式，并得到了所有浏览器的支持。FlexBox 首先是基于 盒装状型 的，Flexible 意味着弹性，使其能适应不同屏幕，补充盒状模型的灵活性。

下图为Flexbox模型图：

FlexBox 把每个视图，都看作一个矩形盒子，拥有内外边距，沿着主轴方向排列，并且，同级的视图之间没有依赖。

和 Auto Layout 类似，FlexBox 采用了描述性的语言去进行布局，而不像 Frame 直接用绝对值坐标进行布局。

>弹性布局的主要思想是让 Flex Container 有能力来改变 Flex Item 的宽度和高度，以填满可用空间（主要是为了容纳所有类型的显示设备和屏幕尺寸）的能力。

最重要的是, FlexBox 布局与方向无关，常规的布局设计缺乏灵活性，无法支持大型和复杂的应用程序（特别是涉及到方向转变，缩放、拉伸和收缩等）。

<h3 id="header1">Flexbox解决了什么？</h3>

> + 方向性 （传统布局方向是从左到右，从上至下）
> + 弹性伸缩 （传统尺寸定义是通过像素等来精确定义）  
> + 元素对齐（可以做到插拔）

从上述几点看来，它似乎完美的解决了iOS原生布局开发效率低的问题，但它会增加页面的嵌套层级关系，在硬件性能饱和的情况下用空间换取开发效率。

<h3 id="header3">iOS基于flexbox的布局</h3>

[yogaKit](https://github.com/facebook/yoga/tree/master/YogaKit)、[FlexBoxLayout](https://github.com/carlSQ/FlexBoxLayout)

<h3 id="header4">YogaKit</h3>

<h4 id="header41">Yoga : 使用跨平台布局引擎<h4>

Yoga 是一个基于 Flexbox 的跨平台布局引擎，能使布局工作更容易。你可以使用 Yoga 作为一个通用的布局系统，来代替 iOS 上的 Auto Layout 或 web 上的 Cascading Style Sheets (CSS)。

最初是 Facebook 在 2014 年推出的一个 CSS 布局的开源库，2016 年改版并更名为 Yoga。Yoga 支持多个平台，包括 Java、C#、C 和 Swift。

库开发者可以集成 Yoga 到他们的布局系统，就如 Facebook 已经集成进了它的两个开源项目：React Native 和 Litho。然而，Yoga 也是一个 iOS 开发者可以直接用来布局视图的框架。

它有以下特点:

1. 兼容Flexbox布局,并增加了特有属性如RTL(自右向左布局)以及AspectRatio(宽高比).

2. 支持Java, C#,Objective-C,C四种语言.

3. 基于C语言的编写,性能更好.

4. 支持流行框架如React Native.

Yoga在iOS端被封装为一个叫做YogaKit的库,通过为UIView的类别增加YGLayout类型的yoga属性来支持UIKit的Yoga布局.

YogaKit是用于iOS开发的，它是基于 Flexbox，它让布局变得更简单。可以用它替代 iOS 的自动布局和 web 的 CSS，也可以将它当成一种通用的布局系统使用。

<h4 id="header42">使用<h4>

1. 通过pod引入

	+ YogaKit 是Swift库，必须动态链接，需要在Podfile种设置 use_frameworks!

2. YogaKit，布局配置是交由 YGLayout 对象来完成

	YogaKit 曝露 YGLayout 作为 UIView 上的一个 Category。这个 Category 添加 configureLayout(block:) 方法到 UIView。将 YGLayout 参数传进闭合块里，并使用这些信息来配置视图的布局属性。

	通过使用所需的 Yoga 属性配置每个参与的视图来构建你的布局。一旦完成，你在根视图的 YGLayout 上调用 applyLayout(preservingOrigin:)。这会计算并应用布局到根视图和子视图。

```C
#import <YogaKit/UIView+Yoga.h>
​
 UIView *container = [[UIView alloc] init];
 [self.contentView addSubview:container];
        
 UIImageView *redView = [UIImageView new];
 redView.backgroundColor = [UIColor redColor];
 [container addSubview:redView];
        
 [self.contentView configureLayoutWithBlock:^(YGLayout * _Nonnull layout) {
    layout.isEnabled = YES;
    layout.height = YGPointValue(88);
    layout.justifyContent = YGJustifySpaceBetween;
    layout.alignItems = YGAlignStretch;
    layout.margin = YGPointValue(0);
    layout.flexDirection = YGFlexDirectionColumn;
            
 }];
        
        [container configureLayoutWithBlock:^(YGLayout * _Nonnull layout) {
            layout.isEnabled = YES;
            layout.margin = YGPointValue(0);
            layout.justifyContent = YGJustifyFlexStart;
            layout.alignItems = YGAlignStretch;
            layout.flexGrow = 1;
            layout.flexDirection = YGFlexDirectionRow;
        }];
        
        [redView configureLayoutWithBlock:^(YGLayout * _Nonnull layout) {
            layout.isEnabled = YES;
            layout.width = YGPointValue(100);
            layout.marginTop = YGPointValue(15);
            layout.marginLeft = YGPointValue(15);
        }];
        
        [self.contentView.yoga applyLayoutPreservingOrigin:NO];
```
​
<h4 id="header43">接入和使用中的问题</h4>

1. 官方维护差强人意: github issue解决和document更新不及时.

2. pod导入有问题

3. The target “YogaKit” contains source code developed with Swift 2.x. Xcode 9 does not support building or migrating Swift 2.x targets.


4. View层级过深

5. UITableViewCell高度不能自适应

___link:___

[iOS 如何使用facebook开源的YogaKit(一)](https://www.jianshu.com/p/c88fa8df0446)

[iOS 如何使用facebook开源的YogaKit(二)](https://www.jianshu.com/p/684e99dad419)

[Yoga 教程](https://juejin.cn/post/6844903491165487118)

[flexbox 语法](http://www.ruanyifeng.com/blog/2015/07/flex-grammar.html)

[iOS上的flexbox布局](https://juejin.cn/post/6844903541295808525)

[iOS Flexbox 布局优化](https://juejin.cn/post/6844903581015867406)