# VMware Horizon 7 Session Enhancement API 简介
VMware Horizon 7 Session Enhancement API 指定了客户端和应用的桌面端如何通过一个VMware Horizon 7 connection进行通信。所有的API交互都是异步的。

任何使用Horizon 7 Session Enhancement API 的软件必须有两部分组件：

- Application应用程序: 这是运行在一个远程桌面上的代码
- Plug-In插件: 这是安装在客户端上的代码

Horizon 7 Session Enhancement API 包涵2种截然不同的API:

- Remote Procedure Call (RPC) API: RPC API提供了一种在运行于远程桌面的Application和运行在客户端上的Plug-In之间通信的异步且回调驱动的通信通道(channel)。RPC API还处理参数的封装处理和解包处理(marshaling and un-marshaling )。
- Overlay API: Overlay API 解决了在客户端上显示已渲染图像的问题。图像是以一种在远程桌面上的本地窗口的形式出现的。

## OpenSSL 事项
Horizon 7 Session Enhancement API动态加载OpenSSL库来实现它的加密特性。如果一个软件的Application和Plug-In 2个组件也要在和Horizon 7 Session Enhancement API相同的进程里，动态加载OpenSSL库 ，你必须遵循下面的规则来避免crash和异常。

1. Plug-In部分必须不能调用`CRYPTO_set_locking_callback()` , `CRYPTO_set_id_callback()` 和 `CRYPTO_set_add_lock_callback()` 函数。因为vmware-remotemks已经调用了这些函数。
1. Application部分必须在加载Horizon 7 Session Enhancement API的库前，设置上述回调。他们必须确保在卸载Horizon 7 Session Enhancement AP库前，这些回调是有效的。
1. 如果代码是在Plug-In部分和Application部分共享的，如果CRYPTO_get_locking_callback返回NULL，你必须调用上述三个回调函数。你必须调用这三个函数来同时设置回调。

## 支持的操作系统
Windows, Linux, and Mac。详见《View Installation》文档。

# VMware Horizon 7 Session Enhancement API 的关键概念

为了有效利用VMware Horizon 7 Session Enhancement API，有必要对Horizon 7 Session Enhancement的一些关键词进行熟悉。

## Connection
Connection是指通过Blast Extreme或PCoIP协议进行的VMware Horizon 7会话。你不能通过Horizon 7 Session Enhancement API换掉一个Connection，但是你可以决定一个Connection的当前状态。如果一个Connection不在连接状态(connected state)，就从API里拿不到事件(action)。你可以通过`OnConnectionStateChanged`回调，用`OnConnectionStateChange`接收到Connection状态变化的通知。你也可以在`VDPService_ChannelInterface` API 里用`GetConnectionState`方法接收到一个Connection的当前状态。
## Channel
Channel代表了一个远程application 和一个本地plug-in间的链接。Channel的状态不必与Connection的状态相同。

你可以通过在通道中注册的`VDPService_ChannelNotifySink`函数来接收通道状态更改的通知。`OnChannelStateChanged`回调会分发状态的变化。你可以在`VDPService_ChannelInterface`中用`GetChannelState`方法来查询一个Channel的当前状态。

## Side Channel

Side Channel代表了一个远程application 和一个本地plug-in间的额外链接。Side Channel属于一个channel对象，并且是通过channel来设置的。一个Side Channel只能在一个channel对象链接完毕后被创建。Side Channel是用于在主channel网络阻塞时，减少application响应时间。比如说，一个application可以使用主channel来传输实时控制信息，并且使用Side Channel来传输大量用户数据。
## Channel Context
Channel Context是一种远程调用参数和返回值的封装体。一个Channel Context持有远程调用接收端所有的信息来确定哪个方法被请求了。与channel  context的交互使用`VDPRPC_ChannelContextInterface`来完成。
## Overlay
Overlay是在另一个窗口上显示的窗口或图像，因此覆盖的图像或窗口看起来像是基础UI的一部分。通常对在本地播放的视频执行此操作，但需要使其看起来像在远程计算机上播放。
## Remote Procedure Call
一个远程程序调用(PRC)是一种在非本地机器上的方法调用。通常，远程机器公开了一套它能够响应的方法，客户端通过一些channel来调用这些方法。一个Invoke调用会启动RPC。
## Sink
sink是一个函数指针结构体，用于与用户代码异步通信。每个API调用都有一组或更多sinks。用户必须注册sink来接收为用户提供重要信息的必要回调。
## Variant
为了简化跨平台通信，与VDP RPC API一起使用的所有参数都包装在VDP_RPC_VARIANT数据类型中。此数据类型包含一个标识符，该标识符指示结构中的数据类型和数据本身。variants的使用通过`VDPRPC_VariantInterface`来完成

# VMware Horizon 7 Session Enhancement 编程流程
一个典型的 Horizon 7 session enhancement编程流程包括了application的初始化，一个plug- in，threads以及一个channel，sink注册，和RPC调用， Overlay API 方法，程序退出。
## Application 初始化
用户控制远程端e Horizon 7 session enhancement系统的启动。启动应用程序后，用户代码将调用`VDPServer_Init`方法并获取`VDP_SERVICE_QUERY_INTERFACE`结构体。然后，用户代码调用`QueryInterface`方法以获取完成其工作所需的所有接口。
## Plugin 初始化
在本地端，Horizon 7 session enhancement系统会初始化插件代码。在`VDPService_PluginInit`调用中，用户代码必须存储对`VDP_SERVICE_QUERY_INTERFACE`结构的传入引用，并使用它来请求所需的所有接口。此时，仅加载用户代码。 与加载的插件匹配的应用程序启动后，将调用`VDPService_PluginCreateInstance`。在此回调中，用户可以返回在每个回调中返回的指针，以便用户代码可以维持其状态。要匹配plugin和application，请调用`VDPService_PluginGetTokenName`方法，然后将返回的字符串与application给出的字符串进行比较。

从`VDPService_PluginCreateInstance`回调返回之前，用户代码必须从`VDPService_ChannelInterface`调用Connect。

> 注意：由于所使用的基础协议的限制，TokenName变量的长度必须小于**16**个字节。

## Sink 注册

要从Horizon 7 session enhancement 系统接收回调，您必须为不同的通知注册sinks。第一个需要注册的sink是`VDPService_ChannelNotifySink`。该sink会通知您connection状态，channel状态的变化以及何时application创建一个对象。有关对象创建的更多信息，请参见下面的**ChannelObject**部分。要注sink，请使用`VDPService_ChannelInterface`中的`RegisterChannelNotifySink`方法。在sink注册之后，您会收到该sink的句柄，可用于注销该sink。在调用Connect之前，必须注册`DPService_ChannelNotifySink`，以确保在channel可用时收到通知。

在注册`DPService_ChannelNotifySink`为后，您很可能不会收到connection状态更改的回调。这是因为到application或plugin启动时，connection很可能处于连接状态。要在执行任何操作之前确认connection处于正确的状态，请使用`GetConnectionState`方法。

除了`VDPService_ChannelNotifySink`，还存在以下sinks：

- VDPRPC_ObjectNotifySink: 这是针对单个channel对象的。
- VDPRPC_RequestCallback: 这用于每个RPC调用的回调。
- VDPOverlayGuest_Sink: 这是用于guest的重要overlay通知。
- VDPOverlayClient_Sink: 这是用于client的重要overlay通知。

## Thread初始化

- 在applicaiton端，主线程是用户调用`VDPService_ServerInit`的线程。
- 在plugin端，主线程是接收到`VDPService_PluginCreateInstance`回调的线程。
- 对于其他线程，必须先调用`ThreadInitialize`，然后再在RPC API或Overlay API中调用任何其他方法。
- 如果不再需要线程，则必须通过调用`ThreadUninitialize`方法来对其进行去初始化(uninitialized)。

## Channel
为了进行通信，application和plugin之间的channel必须处于活动状态。要初始化channel连接，请调用`Connect`方法。必须在每个channel的连接的两侧调用它。 要关闭通道，请调用`Disconnect`方法。

调用`Disconnect`之后，或者每当channel处于断开状态时，都必须使用`DestroyChannelObject`方法释放所有channel对象。如果再次连接Channel，则必须重新创建任何必需的对象。

# 查询接口

查询接口是application和plugin都必须与之交互的一种数据类型。
查询接口数据类型`VDP_SERVICE_QUERY_INTERFACE`是`vdpService.h`中定义的结构体。application和plugin收到对该结构的引用的方式有所不同。该结构体具有两个成员：版本属性和函数指针。 版本属性会通知用户的应用程序哪些API版本可用。 函数指针是用户的代码如何访问系统中的其他API。 函数指针具有以下定义：

```c++
	Bool (*QueryInterface) (const GUID *iid, void *iface);
```

`QueryInterface()`函数获取用户与Horizon 7 Session Enhancement API交互所需的函数。下表列出了Horizon 7 Session Enhancement定义的GUID，以及该GUID返回的功能列表。

![](https://tva1.sinaimg.cn/large/0081Kckwly1gkr5qxcisaj30vb0e6act.jpg)

以下示例代码显示了如何请求接口：

```c++
VDP_SERVICE_QUERY_INTERFACE qi;
VDPService_ChannelInterface ci; 
qi.QueryInterface(GUID_VDPService_ChannelInterface_V1, &ci);
```

# Application

用户启动Application，该Application是在远程桌面上运行的组件。Application启动并加载**vdpService.dll**后，Application将在DLL中调用`VDPService_ServerInit()`。当Application退出时，它必须调用`VDPService_ServerExit()`。 下表描述了两个服务器功能。

![](https://tva1.sinaimg.cn/large/0081Kckwly1gkr5x7xj9cj30vz0bltau.jpg)

以下示例代码显示了Application如何初始化:

```c++
/* program startup (_tWinMain for example) */ VDP_SERVICE_QUERY_INTERFACE qi;
void *channelHandle;
VDPRPC_VariantInterface vi; VDPOverlay_GuestInterface ogi;
/* other interfaces omitted */
VDPService_ServerInit("example" /* token */, &qi, &channelHandle); qi.QueryInterface(GUID_VDPRPC_VariantInterface_V1, &vi); 
qi.QueryInterface(GUID_VDPOverlay_GuestInterface_V1, &ogi);
/* ... */
```

# Plug-In

 Plug-In和Application之间的主要区别在于，Horizon 7将代码加载到客户端上。 因此，用户编译的代码必须位于系统加载的DLL或共享对象中。 插件必须导出以下功能:
 
 ![](https://tva1.sinaimg.cn/large/0081Kckwly1gkr62g6bbtj30vy0gydjl.jpg)
 
# RPC API
使用RPC API，Plug-In和Application可以跨channels进行通信。 在调用RPC API之前，必须执行所有**VDPService**初始化步骤。

## Channel对象
## Invoke
## Variant
## OnInvoke


# Overlay API

# Installation

# Sample Code

