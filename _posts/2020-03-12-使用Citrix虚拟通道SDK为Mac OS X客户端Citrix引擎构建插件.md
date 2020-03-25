---
layout:     post
title:      "使用Citrix虚拟通道SDK为Mac macOS客户端Citrix引擎构建插件"
date:       2020-03-12 16:08:02
author:     "CoderLeonidas"
catalog: true
tags:

- Cocoa
- macOS
- VDI
- 虚拟化技术
- 翻译


---

# 使用Citrix虚拟通道SDK为Mac macOS客户端Citrix引擎构建插件

本文档介绍了如何为macOS客户端引擎编写虚拟通道插件。要为macOS客户端引擎成功构建虚拟通道插件，您将需要一台运行Mac macOS 10.11, 10.12 or 10.1且安装了Xcode  8.3.3的Mac。

对于服务端，您还需要访问基于Windows的计算机，您可以在其中编译将在服务器上运行的应用程序，并且您应该熟悉Windows虚拟通道SDK。

## 虚拟频道SDK的内容

虚拟频道SDK位于一个名为**Virtual Channel SDK**的文件夹中。 此文件夹包含以下部分：

- 在Virtual Channel SDK中，您不应修改四个文件夹，它们分别是 **Documentation**, **Includes**, **Libraries**, 和 **Build Setting**	

- 有三个文件夹包含示例插件，名为**Ping PlugIn Example**, **Over PlugIn Example,** 和 **Mix PlugIn Example**.每个示例文件夹都包含一个完整的项目，该项目旨在包含在Virtual Channel SDK文件夹内的一个文件夹中。
每个示例文件夹都包含一个Xcode项目，例如**Ping.xcodeproj**，一个**Build Settings**文件夹，一个**Info.plist**文件，一个**English.lproj**文件夹，一个**Server items**文件夹，以及该项目的C源文件和头文件。例如，**Ping PlugIn Example**包含四个源文件：`vdping.c`，`vdping.h`，`pingwire.c`和`pingwire.h`。

	- vdping.c/.h: 其中定义了与服务端交互的数据结构体，以及实现了如下函数：
	
	```c
	int DriverOpen( PVD, PVDOPEN, PUINT16 );
	int DriverClose( PVD, PDLLCLOSE, PUINT16 );
	int DriverInfo( PVD, PDLLINFO, PUINT16 );
	int DriverPoll( PVD, PDLLPOLL, PUINT16 );
	int DriverQueryInformation( PVD, PVDQUERYINFORMATION, PUINT16 );
	int DriverSetInformation( PVD, PVDSETINFORMATION, PUINT16 );
	int DriverGetLastError( PVD, PVDLASTERROR );
	```
	是客户端最需要关注的文件。
	
	- pingwire.c/.h: 定义了一些和字节对齐相关的函数，非必要：

	```c
	#if defined(BIGEND) || defined(ALIGNMENT_REQUIRED)
	USHORT Marshall_Write_VDPING_C2H(PVDPING_C2H input, USHORT inputSize);
	USHORT Marshall_Write_PING(PPING input, USHORT inputSize);
	void Marshall_Read_PING(LPVOID input, LPVOID output);
	#endif // BIGEND || ALIGNMENT_REQUIRED
	```
	这些函数用于在客户端引擎上的布局与服务器上的布局之间转换VDPING C2H或PING数据结构中的数据的函数。这些功能在Big endian(大端，认为按照从低地址到高地址的顺序存放数据的高位字节到低位字节，第一个字节是最高位字节)架构(如PowerPC)和需要对齐数据的架构(如ARM)上是必需的。

- 成功构建插件后，该插件的文件夹内将有一个build夹，依次包含三个文件夹Debug，Release和一个以您的项目命名的文件夹，如**Ping.build**。Release文件夹将包含您要发送的插件； 例如名为**Ping.PlugIn**。 Debug文件夹包含客户端的调试版本。

- Server items文件夹包含构建每个示例的服务器副本所必需的源代码和其他文件。
您将需要Windows虚拟通道SDK，其中包含有关如何从这些文件中构建服务器可执行文件的文档。每个文件夹还包含一个文件**ctxping.exe.sample**或类似文件。 这是要在服务器上运行的已编译可执行文件。运行后需要删除后缀`.sample`； 只是为了使这些文件的传输更加容易。

例子插件设置为与Xcode 5.1或更高版本一起使用，并使用macOS 10.9的deployment target。默认情况下，它们会生成可在基于Intel的Mac上运行的代码。 您应该通过复制提供的Xcode示例项目之一来开始开发。


使用Xcode打开每个示例项目时，将在Xcode 工程中下找到以下分组：

- Settings: 它包含一个配置文件**_ProductSettings.xcconfig**，您需要对其进行修改。

- Sources: 它包含插件的所有源文件和头文件。 您可以从示例中的文件开始，但是显然您必须添加自己的功能。

- Resources: 包含**Info.plist**和**InfoPlist.strings(English)**。 更改_ProductSettings.xcconfig后，它们将设置为可以正常工作，但是您可以根据需要进行更改。


-	Virtual Channel SDK:  该文件夹包含Citrix提供的组件。 即文档，头文件，库文件等。 您完全不应修改该组的内容。 实际文件不在您的PlugIn文件夹中，而是在Virtual Channel SDK本身内部。

-	External Frameworks and Libraries: 它包含对任何插件都必需的CoreFoundation框架的引用。 在这里，您将添加插件所需的任何框架。

-	Products: 成功构建后，包含完整构建的插件。

## 创建你自己的插件

Citrix建议您通过使用现有示例项目复制其中一个文件夹来开始创建自己的插件。 假设您想要自己的名为**MyPlugIn**的插件，则需要进行以下更改：

- 复制现有示例插件文件夹之一。 例如，复制“**Ping PlugIn Example**”，并将新文件夹命名为“**MyPlugIn**”。 该副本仍必须位于**Virtual Channel SDK文**件夹中。 文件夹的实际名称无关紧要，只要您对此感到满意即可。


- 在“MyPlugIn”文件夹中，您将找到项目文件**Ping.xcodeproj**。 适当重命名； 例如**MyPlugIn.xcodeproj**。 同样，项目的名称无关紧要。

- 为你的插件决定一个名字， 例如，用MyPlugIn代替Ping。这个名称在用户界面中是可见的，需要添加到模块文件中(请参阅本文档后面的安装插件)。。

- 双击MyPlugIn.xcodeproj，打开您的插件项目。 在“设置”组中，您需要编辑文件_ProductSettings.xcconfig。 将产品名称从Ping更改为MyPlugIn，然后将捆绑包标识符从**com.citrix.ping.samples.PlugIns**更改为针对公司名称定制的**com.mycompany.myPlugIn.PlugIns**。

- 在“Sources”组中，将四个文件**vdping.c**，**vdping.h**，**pingwire.c**和**pingwire.h**重命名为更合适的名称，例如vdmyPlugIn.c，vdmyPlugIn.h，myPlugInwire.c和myPlugInwire.h。 这还将自动重命名硬盘驱动器上的源文件，并适当地更新项目。 适当地更改源文件中的#include指令。 您现在可以构建您的插件。

- 确定插件的虚拟通道名称。 虚拟通道名称是最多**7**个字符的唯一字符串。 保留所有以**CTX**开头的字符串供Citrix使用。 您将在文件vdmyPlugIn.h中找到一个`#define`指令，例如`#define CTXPING_VIRTUAL_CHANNEL_NAME CTXPING`。 您应该更改#define伪指令以及适当使用它的源代码，例如，将其更改为`#define MYPLUGIN_VIRTUAL_CHANNEL_NAME MYPLUGN`。 请记住，虚拟通道名称的长度不能超过7个字符。

## 实现你的插件

为了实现您的插件，您需要遵循Virtual Channel PlugIn for Windows的文档。

## 确保服务器和macOS插件代码之间的数据布局兼容性

您必须确保在服务器和macOS插件之间发送的所有数据结构都使用完全相同的数据布局和相同的字节序。

在macOS或任何其他客户端引擎与服务器之间传输的所有数据结构都需要在编译器指令`#pragma pack(1)`和`#pragma pack()`的包围下进行编译。 Windows编译器和macOS使用的Xcode编译器（包括x86，PowerPC和Arm编译器）都接受这些指令。 这样可以确保尽可能紧密地打包数据结构，但是更重要的是，它可以确保服务器和所有版本的macOS PlugIns上所有数据结构的布局相同。 由于字节序不同，仍然可能存在差异。 来自Ping示例的示例数据结构：


```c
#pragma pack(1)
typedef struct _VDPING_C2H
{
	VD_C2H  Header;
	USHORT  usMaxDataSize; 
	USHORT  usPingCount; 
} VDPING_C2H, * PVDPING_C2H;
#pragma pack()
```


请注意，此示例使用诸如USHORT，ULONG等之类的类型，它们的大小始终相同（分别为2个字节和4个字节），无论编译器为long 和 short选择何种大小。 例如，您不应使用long，因为如果服务器运行64位应用程序且macOS插件为32位，则可能会引起问题，反之亦然。 您必须对8位，16位和32位有符号和无符号数量使用CHAR，SHORT，LONG，UINT8，USHORT和ULONG类型。 请勿使用标准C类型的char，short，int或long来避免服务器与插件之间的兼容性问题。

请注意，在一些示例代码中，您可能会发现**#define**指令，例如`＃define packed_sizeof_VDC2H 48`。这些宏需要在编译器产生与服务器产生不同数据布局的计算机上使用； 在服务器上编译时，此宏将用于数据结构的大小，这可能与在PlugIn中编译的相同结构的大小不同。 这些宏中的数字是手动计数的。 在macOS中不需要这些宏，但是它们仍会起作用。

## 安装插件

要测试您的插件，必须将bundle复制到`~/Library/Application\ Support/Citrix/PlugIns`文件夹中。 （如果该文件夹尚不存在，则必须创建该文件夹；还要注意，插件区分大小写。）


在新的Finder窗口中，打开包含插件的文件夹，然后打开build文件夹。 build文件夹将包含可以忽略的MyPlugIn.build文件夹（或您选择的任何名称），以及名为Debug和Release的文件夹，这取决于您是构建插件的Debug版本还是Release版本。 在内部，您将找到名为MyPlugIn.PlugIn的插件； Debug文件夹还将包含文件MyPlugIn.PlugIn.dSym。 将插件和可选的符号文件拖到上面提到的Plugins文件夹中。

> 注意：
Citrix Viewer最高版本为**11.0.0**的版本不支持名称超过`**12**`个字符的插件，并且它们不会自动在插件名称后附加.PlugIn。 如果要支持这些版本，则需要将插件重命名为MyPlugIn（不带后缀.PlugIn）。 为此，右键单击您的插件，选择“获取信息”，然后取消选择“隐藏扩展名”。 警告后，您可以删除后缀.PlugIn。 如果您的插件名称不超过五个字符，则可以保持不变； 例如Ping.PlugIn，但是您需要使用Ping.PlugIn作为插件名称，而不是Ping。 如果只有晚于11.0.0客户端的客户端，则可以保留插件不变，并使用MyPlugIn或Ping作为插件名称。

接下来，修改`~/Library/Application\ Support/Citrix\ Receiver`文件夹中的**Modules**文件。 

- 在`[ICA 3.0]`部分中，在VirtualDriver行的末尾添加插件的名称，例如Ping.PlugIn。 
- 在该部分中添加一行Ping.PlugIn = On。 
- 添加一个`[Ping.PlugIn]`部分，并将您的Plugin读取的所有设置添加到该部分； 例如，将`PingCount = 3`添加到Ping.PlugIn部分，因为这是Ping示例插件期望的设置。


# Virtual Channel SDK下载链接
[virtual-channel-sdk-Mac下载](https://www.citrix.com/downloads/workspace-app/virtual-channel-sdks/virtual-channel-sdk.html)

[virtual-channel-sdk-windows下载](https://www.citrix.com/downloads/citrix-receiver/virtual-channel-sdks/virtual-channel-sdk.html)