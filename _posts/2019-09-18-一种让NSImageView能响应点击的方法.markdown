---
layout:     post
title:      "一种让NSImageView能响应点击的方法"
// subtitle:   " \"Hello World, Hello Blog\""
date:       2019-09-18 00:38:02
author:     "CoderLeonidas"
catalog: true
tags:
- macOS
- Cocoa

---

NSImageView是无法响应点击事件的，即使你连了IBAction也无法响应。

但是有时我们需要它支持响应点击事件，比如在做聊天消息的Cell时，用ImageView来放一个头像，我们需要点击头像来弹出一个好友的信息框。当然这个例子举的不好，这个头像完全可以用一个NSButton的image来显示，同时又能响应点击。。

在这儿咱们就事论事，谈论怎么给NSImageView加上点击事件。

我们可以子类化NSImageView然后覆盖他的mouseDown：方法：

    #import "LYImageView.h"

    @implementation LYImageView


    //让imageview能够相应点击方法
    - (void)mouseDown:(NSEvent *)event {
        [super mouseDown:event];
        [NSApp sendAction:@selector(clicked:) to:self from:self];
    }

    - (void)clicked:(id)sender {
      NSLog(@"Clicked!!!!");

    //此处可以实现一些默认的点击事件处理，但是对于外部无法自定义点击事件的处理方式
    //可以考虑在外部使用方法交换来实现
    }
    

当然如注释所说，这样做也是有局限性的，外部无法自定义点击事件。

如果有更好的方案，欢迎回复评论，请不吝赐教~
