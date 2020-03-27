用一个很气的属性来解决NSString字符间距在有限宽度内被压缩的问题


标题有些奇怪是不是？总结一下就是：


- 问题: NSString字符间距在有限宽度内被压缩的问题
- 方案：一个属性
- 感受: 很气

那我们用demo来看看这个问题和这个属性是什么，以及它为何很气

代码1：

```objc
- (void)drawRect:(NSRect)dirtyRect {
    [super drawRect:dirtyRect];
    // A truely long string
    NSString *myString = @"There's an old saying goes: 龙生九子，子子不同";

    // A limited drawing rect
    NSRect rect = NSMakeRect(0, 0, 150, 30);
    
    // Stroke this rect
    {
        NSBezierPath *path = [NSBezierPath bezierPathWithRect:rect];
        [[NSColor whiteColor]set];
        [path stroke];
    }
    
    // String attributes
    
    {
        NSMutableDictionary *_attributes = [[NSMutableDictionary alloc] init];
        NSMutableParagraphStyle* paragraphStyle = [NSMutableParagraphStyle new]  ;
        [paragraphStyle setAlignment:NSTextAlignmentLeft];
        [paragraphStyle setLineBreakMode:NSLineBreakByTruncatingMiddle];
//        paragraphStyle.allowsDefaultTighteningForTruncation = NO;

        _attributes[NSParagraphStyleAttributeName] = paragraphStyle;
        
        _attributes[NSForegroundColorAttributeName] = [NSColor redColor];
        
        _attributes[NSFontAttributeName] = [NSFont systemFontOfSize:12];
    }
    
    [myString drawInRect:rect withAttributes:_attributes];
}
```

说明: 字符在此attributes的属性下的物理长度是远大于rect的宽度的，所以它显示不全。显示不全的情况下，我希望它能正常显示，不被压缩，显示不全部分用`...`表示。

代码2：

```objc

````


苹果官方文档:

>
>


验证过程:


demo:


