---
layout:     post
title:      "使用AVPlayer做一个简单的动态壁纸App"
//subtitle:   "持续更新中..."
date:       2019-10-23 21:38:02
author:     "CoderLeonidas"
catalog: true
tags:
- Cocoa
- 音视频

---


# <center>使用AVPlayer做一个简单的动态壁纸App

## 前言 

一直觉得那些动态的桌面壁纸很酷炫，想要自定义却不知道怎么做，所以这次就来尝试着做一个。



动态桌面app在苹果市场上很多见，甚至连mojave中已经自带了一个动态桌面：

![](https://tva1.sinaimg.cn/large/006y8mN6ly1g88frzdn3nj30dw0bptbm.jpg)

动态桌面究竟是什么呢？我猜测应该是个会动的图片，比如gif？mp4？

mojave动态桌面，与以往的普通壁纸相同，新的动态壁纸也都存储在 `/Library/Desktop Pictures` 路径下，文件名分别是 `Mojave.heic` 和 `Solar Gradients.heic`。

![](https://tva1.sinaimg.cn/large/006y8mN6ly1g88g3fmw4zj30jf07xgnv.jpg)

但是考虑到`.heic`格式的动态桌面实现原理过于复杂，我们看看有没有简单一些的方法。

观察App Store上，有很多动态壁纸的App：

![](https://tva1.sinaimg.cn/large/006y8mN6ly1g88g87lh2oj30dw08jdkx.jpg)

我们随便下载一个下来。使用过程我发现，播放过程中不但有图片还有声音，我猜测这就是个短的视频文件，并且循环播放。

于是我去应用程序文件夹下，将该App的报内容显示出来，果然印证了我的猜想：

![](https://tva1.sinaimg.cn/large/006y8mN6ly1g88gbxktvmj30hy06v3zx.jpg)

如图，这个App资源包里有3个`mp4`格式的小视频，而这3个视频也正是这款app免费的动态桌面。

这几个小视频都很短小，每个播放时间在10s左右。

## 思路

于是我就很自然的想到，能不能让一个窗口全屏透明显示，并且在窗口中循环播放小视频，来达到动态桌面的效果呢？

下面附上代码：

```objc


#import "AppDelegate.h"

#import <AVFoundation/AVFoundation.h>

@interface AppDelegate ()
@property (weak) IBOutlet NSView *view;

@property (weak) IBOutlet NSWindow *window;
@property (nonatomic, strong) AVPlayer *player;

@end

@implementation AppDelegate

- (void)applicationDidFinishLaunching:(NSNotification *)aNotification {
    // 设置窗口
    [self setupWindow];
    // 设置窗口为全屏
    [NSApp setPresentationOptions:NSApplicationPresentationFullScreen];
    // 开始播放视频
    [self.player play];
}

- (void)setupWindow {
    NSWindowStyleMask mask = NSWindowStyleMaskBorderless | // 无边框
                             NSWindowStyleMaskResizable | // 可变大小
                             NSWindowStyleMaskFullScreen; // 全屏
    
    self.window.styleMask = mask;
    self.window.level = kCGDesktopWindowLevel; // 窗口层次在桌面之下
    self.view.wantsLayer = YES;
    self.view.layer.backgroundColor = [NSColor clearColor].CGColor; // 设置窗口contentView为透明
    self.window.backgroundColor = [NSColor clearColor]; // 设置窗口为透明
}

- (AVPlayer *)player {
    if (!_player ) {
        _player = [AVPlayer playerWithPlayerItem:[self itemWithIndex:3]];

        AVPlayerLayer *playerLayer = [AVPlayerLayer playerLayerWithPlayer:_player];
        playerLayer.frame = self.view.bounds;
        playerLayer.videoGravity = AVLayerVideoGravityResizeAspect;

        [self.view.layer addSublayer:playerLayer];
        
        [[NSNotificationCenter defaultCenter]addObserver:self selector:@selector(runLoopTheMovie:) name:AVPlayerItemDidPlayToEndTimeNotification object:_player.currentItem];
        
    }
    return _player ;
}

- (AVPlayerItem*)itemWithIndex:(NSUInteger)idx {
    NSString *resName = [NSString stringWithFormat:@"demo%ld", idx];

    NSString *path = [[NSBundle mainBundle] pathForResource:resName ofType:@"mp4"];
    NSURL *url = [NSURL fileURLWithPath:path];
    AVPlayerItem *playerItem = [[AVPlayerItem alloc] initWithURL:url];
    return playerItem;
}

#pragma mark - 接收播放完成的通知，实现循环播放
- (void)runLoopTheMovie:(NSNotification *)notification {
    AVPlayerItem *playerItem = notification.object;
    __weak typeof(self) weakself = self;
    [playerItem seekToTime:kCMTimeZero completionHandler:^(BOOL finished) {
        __strong typeof(self) strongself = weakself;
        [strongself->_player play];
    }];
}


@end

```

思路以下几个要点：

- 程序窗口必须是透明且全屏显示的
- 使用AVPlayer播放视频
- 窗口层次必须在桌面下方，否则会影响到桌面的使用，比如文件的选中等
- 循环播放，通过监听播放完成通知来实现

## 最终效果：


<center>![](https://tva1.sinaimg.cn/large/006y8mN6ly1g88iakmdokg30a006ob29.gif)</center>


## 存在问题

这种方式程序占用的内存比较高，同时循环播放也会比较耗电，这两点还需要后续优化。但是市面上的动态壁纸app貌似都存在这个问题。


# 参考文章

[How to create dynamic wallpapers](https://forums.developer.apple.com/thread/18610#58622)

[macOS Mojave 的动态桌面，可不只是「定时换壁纸」这么简单](https://sspai.com/post/47390)

