---
title: iOS导航栏切换优化
date: 2021-07-01 19:34:24
tags:
categories: IOS
---

### 一、导航栏相关概念：导航栏组件的构成就是一个类似 MVC 的结构

![ios_navbar](https://raw.githubusercontent.com/cocoaGithub/cocoaGithub.github.io/hexo/source/img/ios_navbar_1.png)

1. 我们可以将 UINavigationController 看做是 C，UINavigationBar 看做是 V，而 UIViewController 和 UINavigationItem 组成的 Stack 可以看做是 M。

2. UINavigationController 通过驱动 Stack 中的 UIViewController 的变化来实现 View 层级的变化，也就是 UINavigationBar 的改变。而 UINavigationBar 样式的数据就存储在 UIViewController 的 UINavigationItem 中。

**苹果的UINavigationBar官方文档中有一段对三者的关系的描述非常关键：**


> 当视图控制器在导航过程中，导航控制利用视图控制器提供的navigationItem的属性作为导航控制器的导航栏的model对象。navigationItem默认使用视图控制器的title属性，但是我们也可以通过自定义navigationItem的属性达到完全控制导航栏内容的效果。

**三者的关系：**
1. 每一个导航控制器都会自动生成一个与之对应的导航栏（UINavigationBar）对象；

2. 每个 UIViewController 都有一个属于自己的 UINavigationItem，也就是说它们是一一对应的；当一个新的ViewController加入到导航控制器时，导航栏使用ViewController提供的navigationItem对象作为导航栏的model对象来渲染导航栏；

3. 导航栏会保存导航控制器栈内最上层的两个ViewController的navigationItem对象;

4. ViewController从导航控制器栈移除时，navigationItem会同时从navigationBar的视图栈内移除。

### 二、导航栏组件的生命周期

#### 1.push：ViewControllerA --->   viewControllerB

![push](https://raw.githubusercontent.com/cocoaGithub/cocoaGithub.github.io/hexo/source/img/ios_navbar_2.png)

#### 2. pop：ViewControllerB---> viewControllerA

![pop](https://raw.githubusercontent.com/cocoaGithub/cocoaGithub.github.io/hexo/source/img/ios_navbar_3.png)

三、导航栏隐藏和显示
导航栏里的 Stack 中，每个 ViewController 都可以永久的影响导航栏样式，这种全局性的变化要求我们在实际开发中必须坚持“谁修改，谁复原”的原则，否则就会造成导航栏状态的混乱。

#### 1.根据上面介绍的导航栏状态变化和VC的声明周期，在VC的viewWillAppear、viewWillDisappear分别进行导航栏的修改和恢复。


```C

- (void)viewWillAppear:(BOOL)animated {
    [super viewWillAppear:animated];
    [self.navigationController setNavigationBarHidden:YES animated:animated];
}
​
- (void)viewWillDisappear:(BOOL)animated {
    [super viewWillDisappear:animated];
    [self.navigationController setNavigationBarHidden:NO animated:animated];
}

```

​
问题：

1. 需要在viewWillAppear和viewWillDisappear控制导航栏的地方越来越多，不便于管理和维护

2. 容易出现人为疏忽，导致导航栏未按预期展现

#### 2. hook VC的生命周期节点viewWillAppear、viewWillDisappear集中处理
2.1 基于OC的category功能，导航栏隐藏状态通过扩展属性统⼀注册管理

2.2 hook⻚⾯的⽣命周期节点集中处理导航栏隐藏操作

![页面生命周期](https://raw.githubusercontent.com/cocoaGithub/cocoaGithub.github.io/hexo/source/img/ios_navbar_5.png)

```C
// 具体的一个VC
- (void)viewDidLoad {
   [super viewDidLoad];
    self.cctest_navigationBarHidden = YES;
}
​
// UIViewController + CCtestNavbar(方法交换)
- (void)cctest_viewWillAppear:(BOOL)animated {
    [self viewWillAppear:animated];
    [self.navigationController setNavigationBarHidden:cctest_navigationBarHidden animated:animated];
}
​
- (void)cctest_viewWillDisappear:(BOOL)animated {
    [self viewWillDisappear:animated];
    [self.navigationController setNavigationBarHidden:NO animated:animated];
}
```

效果：实现了导航栏状态统⼀管理,解决了⼤部分场景错位情况

问题：

1. 对于A(隐藏)—> B(隐藏)场景，切换中会有隐藏—显示—隐藏的过度动画；

#### 3. 设置导航控制器的代理,实现代理方法,在将要显示控制器中设置导航栏隐藏和显示


![导航控制器代理](https://raw.githubusercontent.com/cocoaGithub/cocoaGithub.github.io/hexo/source/img/ios_navbar_6.png)


```C
// controller在被加载到navigationController内，将要展现前，判断并控制导航栏的显示或隐藏。
- (void)navigationController:(UINavigationController *)navigationController willShowViewController:(UIViewController *)viewController animated:(BOOL)animated {
    BOOL isHidden = NO;
    isHidden = viewController.klm_navigationBarHidden;
    [self setNavigationBarHidden:isHidden animated:animated];
}
```

效果：

1. 各页面导航栏隐藏状态通过VC的category特性，扩展属性统一注册管理，各页面只需初始化时声明一次

2. 修改导航栏隐藏和显示时机，从页面内生命周期节点迁移至导航容器转场节点

___参考链接：___

http://git.sankuai.com/projects/IOS/repos/sakuikit/browse/Classes/View/ViewController/MTNavigationController

https://docs.sankuai.com/mt/ios/ifst-docs/master/sakuikit/mtnavigationcontroller/

https://www.jianshu.com/p/63b3f21f3d78

https://www.jianshu.com/p/6803ead6df48