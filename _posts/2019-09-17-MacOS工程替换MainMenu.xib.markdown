---
layout:     post
title:      "MacOS工程替换MainMenu.xib"
// subtitle:   " \"Hello World, Hello Blog\""
date:       2019-09-17 18:00:02
author:     "CoderLeonidas"
catalog: true
tags:
- 个人作品
- macOS
- 工具

---

创建了一个新工程，勾选了storyboard。但是工程创建好后就后悔了，不想要storyboard，直接删除了，然后新建了一个MainMenu.xib，在这个xib下添加一个拖一个NSWindow，然后就想把这个window关联到AppDelegate.m里自己定义的一个window属性

`@proporty (weak) IBOutlet NSWindow *window;`

然后死活没法通过拖线关联上。查了一波资料，总算解决了，方法如下：

Xcode默认创建的MainMenu.xib在Interface Builder中的Objects一栏中是有App Delegate这一项，如图：

![屏幕快照 2017-03-06 下午7.31.01.png](http://upload-images.jianshu.io/upload_images/2014369-5a597455df0fb492.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

同时AppDelegate中也已经默认连接好了MainMenu.xib中的NSWindow：

```
@interface AppDelegate ()

@property (weak) IBOutlet NSWindow *window;

@end
```

然而如果自己创建MainMenu.xib的话，这些都是没有的。在Interface Builder中的Objects一栏是这样：

![屏幕快照 2017-03-06 下午7.31.23.png](http://upload-images.jianshu.io/upload_images/2014369-b33c6141409fce02.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


所以我们要增加一个AppDelegate对象上去，并做好和window属性的关联。接下来是具体操作步骤：

1、在Objects Library中找到Object：

![屏幕快照 2017-03-06 下午7.31.33.png](http://upload-images.jianshu.io/upload_images/2014369-52c63dcc7a92b3b9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

2、手动将此Object拖到Interface Builder左侧的Objects一栏内。


3、在右上的Identity Inspector中设置Class为AppDelegate,这样Object名字就变成了AppDelegate，也就是我们需要的AppDelegate对象：

![屏幕快照 2017-03-06 下午7.31.51.png](http://upload-images.jianshu.io/upload_images/2014369-ec24cc6a95aaa697.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

4、拖线，把AppDelegate设置成File’s Owner的delegate

![屏幕快照 2017-03-06 下午7.31.58.png](http://upload-images.jianshu.io/upload_images/2014369-dfadb812b070b2e9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

5、由于之前创建的是勾选StoryBoard的工程，所以target的General的Deployment Info中Main Interface记录的是StoryBoard.xib，此时需要将其改成MainMenu.xib。否则的话Run会崩溃。


![屏幕快照 2017-03-06 下午7.32.08.png](http://upload-images.jianshu.io/upload_images/2014369-b254b33facae5ffe.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

(当然也可以设置Info.plist中的Main nib file base name属性)
![屏幕快照 2017-03-06 下午7.56.50.png](http://upload-images.jianshu.io/upload_images/2014369-43e3a3cc12f6f2d5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


