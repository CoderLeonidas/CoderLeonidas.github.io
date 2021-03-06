---
layout:     post
title:      "ZoomVideo  面试复盘"
subtitle:   "持续更新中..."
date:       2019-10-25 22:38:02
author:     "CoderLeonidas"
catalog: true
tags:
- 面经

---

# <center>ZoomVideo  面试复盘

## 一面

	轮次: 第一轮
	
	面试时间: 2019-10-25 14:00
	
	面试官: 2位
	
	岗位: macOS开发岗
	
	面试时长: 2h

下面是我能回忆起来的面试题:

### 面试题

1. 自我介绍

1. 讲一下内存管理的关键字？strong 和 copy 的区别

1. NSString 用什么关键字？为什么

1. 深拷贝和浅拷贝

1. NSString使用copy关键字，内部是怎么实现的？string本身会copy吗？

1. 使用NSArray 保存weak对象，会有什么问题？

1. 有没有用过MRC？怎么用MRC？

1. MRC和ARC的区别？

1. weak修饰的属性释放后会被变成nil，怎么实现的？

1. KVC平时怎么用的？举个例子

1. KVC一定能修改readonly的变量吗？

1. KVC还有哪些用法？

1. keyPath怎么用的？

1. KVO的实现原理？

1. KVO使用时要注意什么？

1. KVO的观察者如果为weak，会有什么影响？

1. 如何实现多代理？

1. 给一个对象发消息，中间过程是怎样的？

1. 消息转发的几个阶段

1. 设计一个方案，在消息转发的阶段中统一处理掉找不到方法的这种crash

1. 如何实现高效绘制圆角

1. 异步绘制过程中，将生成的image赋值给contents的这种方式，会有什么问题？

1. 前公司负责的模块
	用的语言
	遇到的问题，怎么解决的
	对于该项目当前存在的一些未解决的问题，给出一些解决方案

1. 个人项目展示
	个人项目展示时遇到了crash，面试官给出crashlog，让我分析是什么问题
	项目的设计背景
	项目中的实现细节

1. 是否愿意在MRC环境下进行开发

1. 还有什么问题吗？


### 一面总结:

1. 2位面试官都非常友好，面试过程中一直给我提示，引导我给出更好的回答
2. 面试题都比较简单，在此我就不写答案了，如有需要可以在评论区答复，我会针对性的回答
3. 没有涉及到第三方库、多线程、网路等，可能是我在答题过程中间接涉及到了吧
4. 面试题偏向底层和内存管理，可见他们对内存管理非常重视；而后面问我的一些MRC问题，也印证了这一点
5. 没有涉及到C++问题，有点遗憾，因为C++我已经5年没有写过，为此我好好的准备了一番
6. 要好好准备项目相关问题，不管是前公司的项目还是个人项目，在项目讨论的过程中会碰撞出很多的火花，碰的好，则会给面试官留下好的印象，反之你懂的。。。


### 二面预告:

当时二面面试官在美国，所以二面我将会以视频面试的方式进行。当时是周五，并且美国时间是凌晨4点多了，所以二面通知会在周一给到我。


## 二面


	轮次: 第二轮
	
	面试时间: 2019-10-30 10:30
	
	面试官: 1位
	
	岗位: macOS开发岗
	
	面试时长: 1h



二面结果在周一的下班前大约五点多的时候给到我了，说是周三的早上十点半，进行视频面试。

使用zoom自家的视频会议app，输入会议号，就直接连通了，确实体验非常棒呢。

面试官很眼熟，他也一眼就认出了我。说来惭愧，这是我第三次见到他了，因为在两年多前，我也面试了zoom两次，他是当时的面试官之一。可惜当时我的技术积累十分有限，深度也不够，所以都没能通过。。。

下面是我能会议起来的面试题：

## 二面面试题

1. 前一份工作负责的模块
1. 除了维护性的工作，有没有自己去开发一些新模块或者功能
1. 之前的团队工作流程
1. 怎么做测试的
1. 怎么保证软件质量
1. 如果程序表现与预期的不一致，你会怎么做
2. 前公司做过哪些技术分享
2. 什么时候离职的？这段时间在做什么？
3. 为什么不找好下家就离职了呢？
4. 前公司的忙碌程度？平时下班了都做些什么？
5. 会看一些什么书？
1. 自己的项目介绍
1. 为什么写这个项目，有什么背景
1. 多个模块间如何做通信
1. 是否做过进程间通信，怎么做的
1. 如何与守护进程通信
1. 守护进程的通信，可以用NSNotificationCenter吗？如果不能，可以用什么？
2. C++中的接口是怎么实现的？在OC中有类似接口的概念吗？是什么？
3. 职业规划是什么？
1. 还有什么问题？


### 二面总结

1. zoom的面试官真的都挺和蔼的，网上说的不假，面试过程中不会给你压迫感
2. 二面明显没有涉及到太多技术问题，而其他方面考察的也比较少，基本就是闲聊，很轻松，自由发挥就行
3. 关键词：业余时间安排、前公司表现、测试方法论、自我驱动、职业规划
4. 我觉得二面中最重要的一点是我表现出了对这个岗位的兴趣，当然实时也确实如此。面试官问我为什么呢？我是这么说的：

> - 1 女友在杭州，所以十分想回到杭州
> - 2 对音视频这块一直很感兴趣，而zoom的产品是音视频领域的，并且在业界数一数二
> - 3 zoom的产品的口碑一直很好，体验非常棒，自己很想参与打造一款极致体验的产品并在其中发挥作用
> - 4 很想感受一下美国公司的公司文化


### 三面预告

面试官问我有什么问题的时候，我问下了大概什么时候能给到本轮面试结果。面试官笑笑说，很快就会给到你的。我有点忐忑地说，但愿是个好的结果吧。面试官又笑道，你不用太紧张啊，没事的。面试完了后我就睡觉了。结果在下午五点多的时候，差不多也是临近下班时间，HR电话过来说我二面通过了，我真是超激动啊！三面约在了次日的上午10:30，同样还是视频面试。面试官有2位，分别是HR和杭州区域经理。















