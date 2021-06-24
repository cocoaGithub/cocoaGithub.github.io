---
title: iOS 布局
date: 2018-09-03 19:05:17
tags: 布局
categories: IOS
---

[一、iOS runloop和update cycle](#header1)  
[二、View的更新](#header2)    
[1.约束](#header21)  
[约束更新方法：](#header211)  
[触发约束更新：](#header212)  
[2.布局](#header22)  
[布局更新方法：](#header221)  
[触发布局更新：](#header222)  
[3.显示](#header23)
[显示更新方法：](#header231)
[触发显示更新：](#header232)
[总结](#header4)

<!-- more -->

<h3 id="header1">一、iOS runloop和update cycle</h3>

![runloop](https://raw.githubusercontent.com/cocoaGithub/cocoaGithub.github.io/hexo/source/img/ios_layout_1.png)

ios应用的主runloop负责处理所有的用户输入事件，并触发相应的响应。

所有的用户交互都会被加入到一个事件队列中。下图中的 Application object 会从队列中取出事件并将它们分发到应用中的其他对象上。本质上它会解释这些来自用户的输入事件，然后调用在应用中的 Core objects 相应的处理代码，而这些代码再调用开发者写的代码。

当这些方法调用返回后，控制流回到主 RunLoop 上，然后开始 update cycle（更新周期）

Update cycle 负责布局并且重新渲染视图们

![Update cycle](https://raw.githubusercontent.com/cocoaGithub/cocoaGithub.github.io/hexo/source/img/ios_layout_2.png)

用户交互和布局更新间的延迟几乎不会被用户察觉到。iOS 应用一般以 60 fps 的速度展示动画，就是说每个更新周期只需要 1/60 秒。

这个更新的过程很快，所以用户在和应用交互时感觉不到 UI 中的更新延迟。

但是由于在处理事件和对应 view 重画间存在着一个间隔，RunLoop 中的某时刻的 view 更新可能不是你想要的那样。
 

<h3 id="header1">二、View的更新</h3>

自动布局包含3步来布局和重绘视图。

![自动布局](https://raw.githubusercontent.com/cocoaGithub/cocoaGithub.github.io/hexo/source/img/ios_layout_2.png)


1. 更新约束，系统会计算并给视图设置所有要求的约束。

2. 布局阶段，布局引擎计算视图和子视图的frame并且将他们布局。

3. 显示阶段，重绘视图的内容

布局、显示和约束都遵循着相似的模式，

<h4 id="header21">1.约束</h4>

<h5 id="header211">约束更新方法：</h5>

<table border="1" bordercolor="#a0c6e5" style="border-collapse:collapse;">
	<tr align="left">
	    <th style="color:#000000;background:#EEEEEE;font-size:11pt;font-weight:bold;"> updateConstraints </th>
	    <td><p>在自动布局中动态改变视图约束</p>  <p>updateConstraints() 只应该被重载，绝不要在代码中显式地调用</p>  <p>通常你只应该在 updateConstraints 方法中实现必须要更新的约束。 态的约束应该在视图的初始化方法或者 viewDidLoad() 方法中指定。</p></th>
	</tr >
</table>


<h5 id="header211">触发约束更新：</h5>

<table border="1" bordercolor="a0c6e5" style="border-collapse:collapse;" >
	<tr align="left" >
	    <th style="color:#000000;background:#EEEEEE;font-size:11pt;font-weight:bold;" > 自动触发 </th>
		<td> <p>设置或者解除约束</p> <p>更改约束的优先级或者常量值</p> <p>或者从视图层级中移除一个视图时</p> </td>
	    <td> <p>设置一个内部的标记 “update constarints”</p> <p>这个标记会在下一个更新周期中触发调用 updateConstrains</p> </td>
	</tr >
	<tr>
		<th rowspan="3" style="color:#000000;background:#EEEEEE;font-size:11pt;font-weight:bold;" >手动触发</td>
		<td bordercolor="#FF0000"> setNeedsUpdateConstraints </td>
	    <td> 它通过标记“update constraints”来触发 updateConstraints </td>
	</tr>
	<tr>
		<td> invalidateIntrinsicContentSize </td>
	    <td> <p>自动布局中某些视图拥有 intrinsicContentSize 属性，这是视图根据它的内容得到的自然尺寸。</p>  <p>一个视图的 intrinsicContentSize 通常由所包含的元素的约束决定，但也可以通过重载提供自定义行为。</p> <p>调用 invalidateIntrinsicContentSize() 会设置一个标记表示这个视图的 intrinsicContentSize 已经过期，需要在下一个布局阶段重新计算。</p> </td>
	</tr>
	<tr>
		<td> updateConstraintsIfNeeded </td>
	    <td> <p>它会检查 “update constraints”标记</p> <p>如果它认为这些约束需要被更新，它会立即触发 updateConstraints() ，而不会等到 run loop 的末尾。 </td>
	</tr>
</table>

<h4 id="header22">2.布局</h4>

一个视图的布局指的是它在屏幕上的的大小和位置。每个 view 都有一个 frame 属性，用来表示在父 view 坐标系中的位置和具体的大小。UIView 给你提供了用来通知系统某个 view 布局发生变化的方法，也提供了在 view 布局重新计算后调用的可重写的方法。

<h5 id="header221">布局更新方法：</h5>

<table border="1" bordercolor="#a0c6e5" style="border-collapse:collapse;">
	<tr align="left">
	    <th style="color:#000000;background:#EEEEEE;font-size:11pt;font-weight:bold;"> layoutSubviews </th>
		<td> <p>对视图（view）及其所有子视图（subview）的重新定位和大小调整</p> <p>负责给出当前 view 和每个子 view 的位置和大小</p> <p>开销很大，因为它会在每个子视图上起作用并且调用它们相应的 layoutSubviews 方法</p> <p>系统会在任何它需要重新计算视图的 frame 的时候调用这个方法，所以你应该在需要更新 frame 来重新定位或更改大小时重载它</p> <p>你不应该在代码中显式调用这个方法</p> 
		</td>
	</tr >
</table>

有许多可以在 run loop 的不同时间点触发 layoutSubviews 调用的机制，这些触发机制比直接调用 layoutSubviews 的资源消耗要小得多。

<h5 id="header222">触发布局更新：</h5>

<table border="1" bordercolor="#a0c6e5" style="border-collapse:collapse;">
	<tr align="left">
	    <th style="color:#000000;background:#EEEEEE;font-size:11pt;font-weight:bold;"> 自动刷新触发 </th>
		<td> <p>这些自动通知系统 view 的布局发生变化的方式有：</p> <p>修改 view 的大小</p> <p>新增 subview</p> <p>用户在 UIScrollView 上滚动（layoutSubviews 会在 UIScrollView 和它的父 view 上被调用）</p> <p>用户旋转设备</p> <p>更新视图的 constraints</p>
		</td>
		<td/>
	</tr >
	<tr align="left">
	    <th rowspan="2" style="color:#000000;background:#EEEEEE;font-size:11pt;font-weight:bold;" > 主动刷新触发 </th>
		<td> setNeedsLayout	 </td>
		<td> <p>调用这个方法代表向系统表示视图的布局需要重新计算。</p> <p>setNeedsLayout 方法会立刻执行并返回，但在返回前不会真正更新视图。视图会在下一个 update cycle 中更新，就在系统调用视图们的 layoutSubviews 以及他们的所有子视图的 layoutSubviews 方法的时候。</p> <p>即使从 setNeedsLayout 返回后到视图被重新绘制并布局之间有一段任意时间的间隔，但是这个延迟不会对用户造成影响，因为永远不会长到对界面造成卡顿。</p> </td>
	</tr >
	<tr align="left">
		<td> layoutIfNeeded	 </td>
		<td> <p>layoutIfNeeded 会立即调用 layoutSubviews 方法。</p> 
<p>但是如果你在一个runl oop调用了 layoutIfNeeded 之后，并且没有任何操作向系统表明需要刷新视图，那么就不会调用 layoutsubview</p> </td>
	</tr >
</table>

这些事件会自动给视图打上 “update layout” 标记，告知系统 view 的位置需要被重新计算，继而会自动转化为一个最终的 layoutSubviews 调用

<h4 id="header23">3.显示</h4>

一个视图的显示包含了颜色、文本、图片和 Core Graphics 绘制等视图属性，不包括其本身和子视图的大小和位置。和布局的方法类似，显示也有触发更新的方法，它们由系统在检测到更新时被自动调用，或者我们可以手动调用直接刷新。

<h5 id="header231">显示更新方法：</h5>

<table border="1" bordercolor="#a0c6e5" style="border-collapse:collapse;">
	<tr align="left">
	    <th style="color:#000000;background:#EEEEEE;font-size:11pt;font-weight:bold;" > drawRect </th>
		<td> <p>draw 方法不会触发后续对视图的子视图方法的调用</p> <p>你不应该直接调用 draw 方法，而应该通过调用触发方法，让系统在 run loop 中的不同结点自动调用</p>
		</td>
	</tr >
</table>

<h5 id="header232">触发显示更新：</h5>

<table border="1" bordercolor="#a0c6e5" style="border-collapse:collapse;">
	<tr align="left">
	    <th rowspan="2" style="color:#000000;background:#EEEEEE;font-size:11pt;font-weight:bold;" > setNeedsDisplay </th>
		<td> <p>它会给有内容更新的视图设置一个内部的标记，但在视图重绘之前就会返回。</p> <p>在下一个 update cycle 中，系统会遍历所有已标标记的视图，并调用它们的 draw 方法。
		</td>
	</tr >
	<tr align="left">
		<td> <p>如果你只想在下次更新时重绘部分视图，你可以调用 setNeedsDisplay(_:)，并把需要重绘的矩形部分传进去（setNeedsDisplayInRect in OC)。</p> <p>大部分时候，在视图中更新任何 UI 组件都会把相应的视图标记为“dirty”，通过设置视图“内部更新标记”，而不需要显式的 setNeedsDisplay 调用
		</td>
	</tr >
</table>

视图的显示方法里没有类似布局中的 layoutIfNeeded 这样可以触发立即更新的方法。通常情况下等到下一个更新周期再重新绘制视图也无所谓。

<h3 id="header1">总结</h3>

下面的表格列出了任意组件会怎样更新及其对应方法。

<table border="1" bordercolor="#a0c6e5" style="border-collapse:collapse;">
	<tr align="left">
	    <th style="color:#000000;background:#EEEEEE;font-size:11pt;font-weight:bold;" > Method purpose </th>
		<th style="color:#000000;background:#EEEEEE;font-size:11pt;font-weight:bold;" > Layout（布局） </th>
		<th style="color:#000000;background:#EEEEEE;font-size:11pt;font-weight:bold;" > Display（显示） </th>
		<th style="color:#000000;background:#EEEEEE;font-size:11pt;font-weight:bold;" > Constraint（约束） </th>
	</tr >
	<tr align="left">
	    <td> implement updates（override，don't call explicitly）实际更新方法 </td>
		<td> layoutSubviews </td>
		<td> draw </td>
		<td> updateConstraints </td>
	</tr >
	<tr align="left">
	    <td> Explicitly mark view as needing update on next update cycle显式标记下一个周期需要更新 </td>
		<td> setNeedsLayout </td>
		<td> setNeedsDisplay  </td>
		<td> <p>setNeedsUpdateConstraints</p> <p>invalidateInstrinsicContentSize</p> </td>
	</tr >
	<tr align="left">
	    <td> update immediately if View is marked as “dirty”立即执行更新 </td>
		<td> layoutIfNeeded </td>
		<td> </td>
		<td> updateConstraintsIfNeeded </td>
	</tr >
	<tr align="left">
	    <td> Actions that implicitly cause View to be updated隐式标记下一个周期需要更新 </td>
		<td> <p>addSubviews</p> <p>Resizing 视图</p> <p>通过setFrame改变视图的bounds</p> <p>用户滑动UIScrollView</p> <p>用户旋转设备</p> </td>
		<td> 改变视图的颜色、文本、图片和 Core Graphics 绘制等视图属性 </td>
		<td> <p>设置或者解除约束</p> <p>更改约束的优先级或者常量值</p> <p>从视图层级中移除一个视图</p> </td>
	</tr >
</table>

三、 update cycle 和 event loop 之间的交互

你可以在 run loop 中的任意一点显式地调用 layoutIfNeeded 或者 updateConstraintsIfNeeded，需要记住，这开销会很大。在循环的末端是 update cycle，如果视图被设置了特定的 “update constraints”，“update layout” 或者 “needs display” 标记，在这节点会更新约束、布局以及展示。一旦这些更新结束，runloop 会重新启动

links：

[揭秘ios布局](https://juejin.cn/post/6844903567610871816)
