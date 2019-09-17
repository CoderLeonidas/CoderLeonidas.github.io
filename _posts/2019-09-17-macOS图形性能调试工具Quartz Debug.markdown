---
layout:     post
title:      "macOS图形性能调试工具Quartz Debug"
// subtitle:   " \"Hello World, Hello Blog\""
date:       2019-09-17 18:50:02
author:     "CoderLeonidas"
catalog: true
tags:
- macOS
- Cocoa
- 图形
---


在macOS上有个专门来调试图形性能的工具，叫做，Quartz Debug 。它存在于一个叫做Additional_Tools_for_Xcode的扩展工具包中，每个xcode版本都有对应的扩展工具包dmg，如xcode10.1，需要下载Additional_Tools_for_Xcode_10.1，可以在[这里](https://developer.apple.com/download/more/)下载到。本文主要内容参考《[iOS和macOS性能优化：Cocoa、Cocoa Touch、Objective-C和Swift](http://www.broadview.com.cn/book/4920)》，Quartz Debug提供了许多工具来帮助您调试应用程序中与图形相关的问题。下面讨论Quartz调试的特性。

## Quartz Debug Settings

![image.png](https://tva1.sinaimg.cn/large/006y8mN6gy1g72pgm6aouj30ce0pp760.jpg)


使用此面板切换常用加速和调试选项。提供以下选择:

*   Enable Quartz Debug

启用/禁用所有选项。

*   Disable 2D Acceleration

禁用所有2D加速。


`
所谓2D加速是指2D图形硬件加速，即开启GPU。强制进行GPU渲染，指的是在CPU性能不足或者想要节省CPU资源的情况下，强制系统使用GPU对系统和软件的UI进行渲染，由于在图形方面GPU相比CPU具有天然优势，所以此情况下使用GPU渲染会得到更加流畅的界面体验；
`


*   Autoflush drawing

每次绘图操作后刷新内容。
会关闭合并内存的访问模式，所以每个绘制操作都会直接展示在屏幕上(如果开启Flash identical screen updates和Flash screen updates2个选项，可能会造成闪烁)，这会产生更多的干扰信息，但是这样会将绘制过程划分的更细，展示的矩形越多，就越能让开发者了解系统是如何绘制的以及绘制过程发生了哪些变化。

*   Flash screen updates

在屏幕区域更新之前用黄色高亮显示(正常)。DisableUpdate下的区域被涂成橙色。
该选项能让你区分出那些刷新次数过于频繁的地方。打开该选项可能会产生一些干扰数据。

*   Flash identical screen updates

用红色高亮显示冗余屏幕更新。冗余更新是指重新绘制未更改像素的情况。

开启该选项后，Quartz会用红色矩形块标注屏幕中重复刷新相同内容的区域，这表示红色区块的绘制操作是多余的，应该被删除。


*   No delay after flash

移除闪烁屏幕更新后的延迟。

*   Show tracking rectangles

跟踪矩形用绿色表示。活动跟踪矩形的轮廓是红色的。

*   Vertical Sync

切换垂直波束同步开关的窗口服务器屏幕更新。

## UI Resolution

UI解析窗口允许控制用于用户界面的当前缩放因子。设置完成后，将立即向Dock应用新的分辨率，但是应用程序需要重新启动才能获取新的设置。

## Framemeter

![](https://tva1.sinaimg.cn/large/006y8mN6gy1g72phwve73j30fl0ddwex.jpg)

帧表显示每秒屏幕更新的次数。还显示了CPU使用情况。帧表输出也可以在Quartz Debug Dock图标中获得。

## Dock Icon

可以将dock图标配置为显示帧表和石英状态。右键单击dock图标或选择Tools -> dock来配置这些选项

*   Show Framemeter History

帧速率绘制在每秒更新10次的图形上，并显示在dock中应用程序图标的位置。该图由当前FPS读取值着色，红线表示更新速率为0……一条黄线31……一条绿线61……90 fps。


---
应该注意，随着Quartz Debug的运行，应用程序与平常的运行状态有一点区别，绘制这些额外的矩形会造成不小的开销，你甚至可以感觉到屏幕在刷新的过程中有一些延迟。开启一个Flash选项，然后尝试拖拽窗口，这时不仅能看到有很多闪烁效果，拖拽的过程也变的很迟钝。关闭延迟能让性能恢复正常，但闪烁会导致肉眼难以识别屏幕上的情况。

***

参考资料：
《[iOS和macOS性能优化：Cocoa、Cocoa Touch、Objective-C和Swift](http://www.broadview.com.cn/book/4920)》

《Quartz Debug 4.0 Help(位于Quartz Debug 的help中)》
