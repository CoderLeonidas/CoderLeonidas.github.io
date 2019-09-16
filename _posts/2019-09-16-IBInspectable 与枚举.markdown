---
layout:     post
title:      "IBInspectable与枚举"
subtitle:   " \"Hello World, Hello Blog\""
date:       2019-09-16 12:00:00
author:     "CoderLeonidas"
catalog: true
tags:
- Cocoa
- XCode
---

在使用`IBInspectable`定义可视化属性时，对于枚举类型，是没法在xib上可视化的。代码如下：

```

#import <Cocoa/Cocoa.h>

NS_ASSUME_NONNULL_BEGIN
typedef enum : NSUInteger {
EMBorderViewTypeNone,
EMBorderViewTypeDash,  ///< 虚线边界线
EMBorderViewTypeSolid,  ///< 实线边界线
} EMBorderLineType;

typedef NS_OPTIONS(NSUInteger, EMBorderMode) {
EMBorderModeNone   = 1 << 0,  ///< 没有边界线
EMBorderModeLeft   = 1 << 1,  ///< 有左边界线
EMBorderModeRight  = 1 << 2,  ///< 有右边界线
EMBorderModeTop    = 1 << 3,  ///< 有上边界线
EMBorderModeBottom = 1 << 4,  ///< 有下边界线
EMBorderModeAll    = 1 << 5,  ///< 有全部边界线
};

IB_DESIGNABLE

@interface EMBorderView : NSView
@property (nonatomic,strong) IBInspectable NSColor *lineColor;
@property (nonatomic, assign) IBInspectable EMBorderLineType borderLineType;  ///< 边界线类型，对应EMBorderLineType枚举值
@property (nonatomic, assign) IBInspectable EMBorderMode borderMode; ///< 边界线模式，对应EMBorderMode枚举值
@property (nonatomic, assign) IBInspectable CGFloat lineWidth; ///< 线宽

- (instancetype)initWithLineType:(EMBorderLineType)lineType mode:(EMBorderMode)mode;

@end

NS_ASSUME_NONNULL_END
```

这段代码中，只有lineColor、lineWidth是可以可视化的：

![clip_6.png](https://upload-images.jianshu.io/upload_images/2014369-9b2c6528fa4786c8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/719)


而2个枚举属性borderLineType和borderMode无法在属性监视器上显示。
查了下资料，对于`IBInspectable`关键字支持的属性类型，发现它仅支持以下几种基础数据类型的属性：

```
Int
CGFloat
Double
String
Bool
CGPoint
CGSize
CGRect
UIColor
NSColor
UIImage
NSImage
```

我们知道枚举本质也就是NSIngeger或NSUInteger，所以为了让这2个枚举属性可视化，我们可以将属性这样定义：

```
@property (nonatomic, assign) IBInspectable NSUInteger borderLineType;  ///< 边界线类型，对应EMBorderLineType枚举值
@property (nonatomic, assign) IBInspectable NSUInteger borderMode; ///< 边界线模式，对应EMBorderMode枚举值
```
同时注意setter方法中也需要将其类型改为NSUInteger，如：

```
- (void)setBorderMode:(NSUInteger)borderMode {
if (_borderMode == borderMode) {
return;
}
_borderMode = borderMode;
[self refresh];
}
```

这样我们就能在属性监视器上看到我们定义的属性：
![image.png](https://upload-images.jianshu.io/upload_images/2014369-6709d2009c666c20.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


当然，这个方法的缺点就是，在属性监视器中设置枚举类型属性时，要自己对所需的枚举值做运算。这或许不是最好的方法，如果大家有什么更好的方法来优雅的使用`IBInspectable`定义可视化属性，欢迎大家讨论不吝赐教~

## 参考链接：
[@IBInspectable with enum?](https://stackoverflow.com/questions/28749598/ibinspectable-with-enum)




