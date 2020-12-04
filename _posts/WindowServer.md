WindowServer：显示合成器和输入事件路由

WindowServer，有时候是多个空格的Window Server，它是一组处理窗口管理，合成窗口到显示图像，并为应用程序的事件路由的内部相互通信的服务。作为GUIde一个关键部分，它被启动以支持登录窗口，一直运行知道用户登出。多年来，他在不同的系统间移植：最初是Quartz Compositor，后来是CoreGraphics的一部分，最近是SkyLight的一部分。在Catalina中，可以在` /System/Library/PrivateFrameworks/SkyLight.framework/Resources`中找到，以及指针/光标的支持。




参考文章:

[WindowServer: display compositor and input event router](https://eclecticlight.co/2020/06/08/windowserver-display-compositor-and-input-event-router/)