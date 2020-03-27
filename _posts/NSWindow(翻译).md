NSWindow笔记

**NSWindow**是应用程序在屏幕上显示的窗口。


# 定义：

```objc
@interface NSWindow : NSResponder
```

可见NSWindow 继承自`NSResponder`，所以是响应者链中的一环。

# 概述：

一个NSWindow对象最多对应一个屏幕窗口。 窗口的两个主要功能是：

- 提供一个可以放置**视图**的区域
- 接受用户通过使用鼠标和键盘进行的操作引发的**事件**并将其分配给适当的视图。

>
尽管NSWindow类从**NSResponder**继承了**NSCoding**协议，但该类不支持编码。 存在对archivers的旧式支持，但是不建议使用它，并且可能无法使用。 任何尝试使用c编码对象归档或取消归档窗口对象的操作都会引发**NSInvalidArgumentException**异常。

# 主题

关于window的主题特别多。下面仅列出每个主题的标题，详细内容(属性、****方法)请到[NSWindow官方文档](https://developer.apple.com/documentation/appkit/nswindowtabgroup?language=objc)去看。


### 创建一个window

###管理window的行为

###配置window的内容

###配置window的外观

###访问window信息

###获取布局信息

###管理多个window

###管理多个表单

###调整windows大小

###调整内容大小

###管理window的layers

###管理window可见性和遮挡状态

###在用户默认设置下管理窗口frame

###管理关键状态

###管理主状态

###管理工具条

###管理附属的windows

###管理默认按钮

###管理字段编辑器

###管理window菜单

###管理鼠标矩形

###管理标题栏

###管理toolbar-titlebar区域

###管理窗口标签

###管理tooltips

###事件处理

###管理响应者

###管理key view loop

###处理键盘事件

###处理鼠标事件

###处理window还原

###绘制window

###window动画

###window更新

###拖拽项目

###访问编辑状态

###转换座标

###管理标题

###访问屏幕信息

###移动window

###关闭window

###最小化window

###获得dock tile

###打印window

###提供服务

###触发基于约束的布局

###调试基于约束的布局

###基于约束的布局

###与Carbon框架一起工作

###处理window深度

###获取有关脚本属性的信息

###设置脚本属性

###处理脚本命令

###使用有序索引

###常量

###通知

###实例属性

###实例方法

# 原文
[NSWindow](https://developer.apple.com/documentation/appkit/nswindowtabgroup?language=objc)
