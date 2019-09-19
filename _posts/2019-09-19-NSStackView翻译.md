---
layout:     post
title:      "NSStackView翻译"
// subtitle:   " \"Hello World, Hello Blog\""
date:       2019-09-19 22:38:02
author:     "CoderLeonidas"
catalog: true
tags:
- 翻译
- Cocoa

---

# NSStackView翻译
 

>stackView可以水平或垂直方向来管理一组视图，自动更新它们放置的位置并且在窗口大小变化时变化大小。


## 概述
stackview使用自动布局（系统的自动布局特性）来根据你的要求管理和对齐一组视图。要想高效使用stackview，你需要理解自动布局约束的基础，这在《Auto Layout Guide》中有描述。



### stackview的基本特性
stackview支持水平和垂直方向的布局，并且在窗口大小变化和Cocoa动画时自动发生作用。你可以在运行时简单配置stackview的内容。也就是说，当你在IB中创建和配置stackview后你可以动态的添加或者删除视图，而不用显式地使用布局约束。比如说，如果你给一个stackview设置了3个checkbox，并且动态地增加第四个，stackview会根据自身属性的设置，自动根据需要添加约束。新的checkbox会从stackview那获得动态的布局设置(约束)。

stackview是可以嵌套的，一个stackview在另一个stackview的一组视图中也是一个有效的元素。



>###### 要点 
>>不要对stackview的私有视图添加自视图或者约束。stackview的私有视图可能会在未来macOS版本中改变，并且不确保会支持NSCoder编码解码(encoded or decoded).

### 布局方向和重力区域
一个stackview有3个所谓的重力区域，每个区域分别标识堆栈视图布局的一部分。一个水平方向的stackview(默认类型)
，有一个前向的，一个中间的(不一定是左、中、右)，一个后向的重力区域。这些区域的顺序取决于stackview的userInterfaceLayoutDirection属性(继承自NSView)。在左至右的语言中，水平stackview的前向重力区域位于左侧。为了独立于语言强制从左到右布局，需要在你的stackview实例中显式的调用userInterfaceLayoutDirection方法。

对于垂直布局，就是用orientation属性为NSUserInterfaceLayoutOrientationVertical(来自枚举NSUserInterfaceLayoutOrientation)。在一个垂直的stackview中，重力区域的分是上、中、下。


### 视图分离和隐藏

stackview会自动分离或者重新固定它的视图以响应布局的变化，比如由用户发起的窗口大小重变化，或者在相同的视图层次结构中调整或重新定位另一个视图。处于分离状态的视图不会出现在stackview的视图层级中，但是它仍然消耗占用内存。

为了允许视图分离，需要设置stackview所谓的clipping resistance(剪切阻力)的值低于默认值NSLayoutPriorityRequired。详情见setClippingResistancePriority:forOrientation:方法。

你可以影响哪个视图首先分离(并且最后重新连接)。通过设置所谓的visibility属性给每个你想要指定分离顺序的视图。一个visibility属性值较低的视图会比一个值较高的视图先 分离，并且重连接也在其之后。详情见NSStackViewVisibilityPriority枚举和setVisibilityPriority:forView: 方法。

要显示的将一个视图从stackview中分离，可以调用setVisibilityPriority:forView: 方法并给一个NSStackViewVisibilityPriorityNotVisible值。要显式重连接一个视图到stackview，还是调用这个方法，并给一个NSStackViewVisibilityPriorityMustHold值。如果你隐藏一个属于stackview的视图(通过调用视图的hidden方法，并给YES值)，视图并不会从stackview分离。他不再可见，也不再接受输入时间，但是它任然是视图层级的一部分，并会继续参与自动布局。

当一个视图将要被分离或者被重连接时，系统会调用stackview的代理方法，来给你时机在那瞬间运行代码。详情见NSStackViewDelegate。



## 主题

### 初始化一个stackview
#### `+ stackViewWithViews:`

用一个特定的视图数组来创建并返回一个stackview

---

### 响应stack相关的变化

#### `delegate`
stackview 的代理对象
#### `NSStackViewDelegate`

要想设置一个自定义类来响应视图的分离或重连解到stackview，设置这个自定义类遵循NSStackViewDelegate协议。然后设置stackview的delegata为你的自定义类对象。

---

### 创建和设置一个stackview
#### `- setViews:inGravity:`

给一个stackview的特定重力区域设置一组视图，替换之前这个重力区域之前的所有视图。


#### `alignment`

stackview内部的视图对齐方式

#### `orientation`

stackview的水平或者垂直布局方向

#### `spacing`

stackview中相邻视图的最小间距，以像素点为单位

#### `edgeInsets`

stackview内部的填充间距，环绕它的所有view，以像素点为单位

---

### 设置stackview的动态行为
#### `- setClippingResistancePriority:forOrientation:`

当自动布局试图减少stackview的大小时，设置自动布局优先级，以便在stackview中防止子视图的裁剪。

#### `- setHuggingPriority:forOrientation:`

设置堆栈视图的自动布局优先级，以最小化它的大小，为指定的用户界面坐标轴。

---

### 添加或移除子视图
#### `- addView:inGravity:`

在当前stackview重力区域后面，添加一个视图到重力区域

#### `- insertView:atIndex:inGravity:`

在指定下标位置天机一个视图到stackview重力区域

#### `- removeView:`

从stackview中移除一个特定的视图

---

### 在stackview中设置视图
#### `- customSpacingAfterView:`

返回特定视图和它后一个视图的间距，以像素点为单位

#### `- setCustomSpacing:afterView:`

指定特定视图和它后一个视图的间距，以像素点为单位

#### `- visibilityPriorityForView:`

返回特定视图的可见优先级( visibility priority )

#### `- setVisibilityPriority:forView:`

设置一个视图的自动布局优先级以在自动布局减少stackview大小时保持和stackview的连接

---

### 检查stackview
#### `views`

stackview拥有的一组视图

#### `- viewsInGravity:`

返回stackview中特定重力区域的视图数组

#### `detachedViews`

返回已经从stackview所有重力区域分离的视图数组


#### `- clippingResistancePriorityForOrientation:`

返回当自动布局试图减少stackview的大小时，为在stackview中拒绝裁剪视图的自动布局优先级

#### `- huggingPriorityForOrientation:`

返回stackview为一个用户界面坐标轴最小化大小来尽可能适应所有包含的视图时的自动布局优先级

---

### 常量
#### `NSUserInterfaceLayoutOrientation`

两个stackview布局方向，对于剪切阻力和hugging优先级，是两个用户界面坐标轴
#### `NSStackViewGravity`

在stackview中可用的各种重力区域

#### `NSStackViewVisibilityPriority`

为视图在stackview中保持连接的各种自动布局优先级。

#### `NSStackViewSpacingUseDefault`

在stackview中的每个视图后面指定默认间距的标志


原文链接 

https://developer.apple.com/library/mac/documentation/AppKit/Reference/NSStackView_Class/Chapters/Reference.html