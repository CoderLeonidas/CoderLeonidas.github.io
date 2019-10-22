---
layout:     post
title:      "CoreFoundation框架 概述"
subtitle:   "持续更新中..."
date:       2019-09-28 22:00:02
author:     "CoderLeonidas"
catalog: true
tags:
- Cocoa
---


# <center>CoreFoundation框架 概述

CF是由C语言实现的，而不是Objective-C，所以如果用到了CF，就需要手动管理内存，ARC是无能为力的。当然因为CF和Foundation之间的友好关系，它们之间的管理权也是可以移交的，这个后面再说。
Core Foundation框架(CoreFoundation.framework) 是一组C语言接口，它们为iOS应用程序提供基本数据管理和服务功能。


> Access low-level functions, primitive data types, and various collection types that are bridged seamlessly with the Foundation framework.


Core Foundation是一个框架，可提供对应用程序服务，应用程序环境以及应用程序本身有用的基本软件服务。 Core Foundation还提供了常见数据类型的抽象，通过Unicode字符串存储促进了国际化，并提供了一系列实用程序，例如插件支持，XML属性列表，URL资源访问和首选项。

## Utilities 工具类

### 基础工具类

Core Foundation定义了许多其他符号，这些符号可被许多不同的不透明类型（例如CFIndex）使用，或整体上应用于Core Foundation（例如kCFCoreFoundationVersionNumber）。 这些符号一起收集并记录在这里。

#### CFComparatorFunction
#### CFRangeMake
#### CFIndex
#### CFOptionFlags
#### CFRange
#### CFComparisonResult


### 字节序工具类

处理跨平台传输或共享的二进制数据时，您需要关注每个平台如何存储数值。 平台以大端或小端格式存储值。 在PowerPC机器等大端机器上，值首先以最高有效字节的形式存储在内存中。 在小字节序的计算机（例如奔腾计算机）上，值首先以最低有效字节存储。 如果其中一台计算机未正确转换以不同格式传输到平台的多字节值，则会被误解。

您可以使用`CFByteOrderGetCurrent`函数标识当前平台的本机格式。 使用`CFSwapInt32BigToHost`和`CFConvertFloat32HostToSwapped`之类的函数在不同的字节顺序格式之间转换值。

TODO 

### CoreFoundation URL 访问工具类
Core Foundation URL Access Utilities为您提供创建，读取，更新或删除URL资源的与系统无关的便捷方法。

给定一个保存文件或http URL的CFURL对象，您可以使用CFURLCreateDataAndPropertiesFromResource函数读取资源的数据。 您可以使用CFURLWriteDataAndPropertiesToResource函数将数据写入URL资源，可能会创建一个新文件。 最后，您可以使用CFURLDestroyResource函数销毁或删除URL指向的资源。

TODO

### 偏好工具类

几个函数将首选项值作为Core Foundation属性列表对象返回。 您可以使用函数CFGetTypeID来确定值的类型。 有关属性列表的更多信息，请参见Core Foundation的属性列表编程主题。

TODO

### 套接字名称服务工具类

名称服务器功能当前在macOS中无法使用。

### 时间工具类

Core Foundation以秒为单位测量时间。 基本数据类型是CFTimeInterval，它测量两次之间的秒差。 固定时间或日期由CFAbsoluteTime数据类型定义，CFAbsoluteTime数据类型用于测量特定日期和格林尼治标准时间1月1日00:00:00的绝对参考日期之间的时间间隔。

CFGregorianDate结构表示公历的绝对时间。 诸如CFAbsoluteTimeGetGregorianDate之类的函数使用CFTimeZone对象来获取特定时区中的本地时间。

CFDate不透明类型将绝对时间包装到基于CFType的对象中，从而允许您将时间对象放入集合和属性列表中，并由Core Foundation的其他面向对象的部分处理。

TODO

## Opaque Types 不透明类型


### CFAllocator

CFAllocator是一种不透明类型，可为您分配和取消分配内存。 您无需直接为Core Foundation对象分配，重新分配或取消分配内存-几乎不需要。 您将CFAllocator对象传递给创建对象的函数。 这些函数的名称中都嵌入了“创建”，例如CFStringCreateWithPascalString。 创建函数使用分配器为其创建的对象分配内存。


### CFArray
### CFAttributedString

### CFBag

CFBag及其派生的可变类型CFMutableBag管理称为袋的值的非顺序集合，其中可以有重复的值。 CFBag创建静态包，而CFMutableBag创建动态包。
当元素的顺序并不重要并且考虑测试集合中是否包含值的性能时，可以使用袋或集合代替数组。


### CFBinaryHeap

CFBinaryHeap实现了一个容器，该容器存储使用二进制搜索算法排序的值。 所有二进制堆都是可变的。 没有单独的不变品种。 二进制堆可以用作优先级队列。

### CFBitVector

CFBitVector及其派生的可变类型CFMutableBitVector管理0或1的位值的有序集合。CFBitVector创建静态位向量，而CFMutableBitVector创建动态位向量。


### CFBoolean
### CFBundle
### CFCalendar
### CFCharacterSet
### CFData
### CFDate
### CFDateFormatter
### CFDictionary
### CFError
### CFFileDescriptor

### CFLocale
诸如归类和文本边界确定之类的Unicode操作可能会受到特定语言或地区的约定的影响。 CFLocale对象为对语言环境敏感的操作指定特定于语言或特定于区域的信息。


### CFMachPort

CFMachPort对象是本机Mach端口（mach_port_t）的包装。 多个端口是macOS内核的本地通信通道。
CFMachPort不提供发送消息的功能，因此，如果需要侦听通过其他方式获得的Mach端口，则主要使用CFMachPort对象。 当消息到达端口或端口无效时（例如本机端口死亡），您可以获取回调。
要侦听消息，您需要使用CFMachPortCreateRunLoopSource创建一个运行循环源，并使用CFRunLoopAddSource将其添加到运行循环中。

> 重要
> 
> 如果要断开连接，则必须在释放运行循环源和Mach端口对象之前使端口无效（使用CFMachPortInvalidate）。
> 

要发送数据，您必须将Mach API与本机Mach端口一起使用，此处不再赘述。 另外，您可以使用CFMessagePort对象，该对象可以发送任意数据。
马赫端口仅支持本地计算机上的通信。 对于网络通信，必须使用CFSocket对象。


### CFMessagePort

和CFMachPort 类似

CFMessagePort对象提供了一个通信通道，可以在本地计算机上的多个线程或进程之间传输任意数据。
您可以使用CFMessagePortCreateLocal创建本地消息端口，并在创建时或稍后使用CFMessagePortSetName为其命名，以使其可用于其他进程。 然后其他进程使用CFMessagePortCreateRemote连接到它，并指定端口的名称。
要侦听消息，您需要使用CFMessagePortCreateRunLoopSource创建一个运行循环源，并使用CFRunLoopAddSource将其添加到运行循环中。


### CFMutableArray
### CFMutableAttributedString
### CFMutableBag
### CFMutableBitVector
### CFMutableCharacterSet
### CFMutableData
### CFMutableDictionary
### CFMutableSet
### CFMutableString
### CFNotificationCenter
### CFNull
### CFNumber
### CFNumberFormatter

### CFPlugIn

CFPlugIn提供了用于应用程序扩展的标准体系结构。 借助CFPlugIn，您可以将应用程序设计为宿主框架，该框架使用称为插件的一组可执行代码模块来提供某些定义明确的功能区域。 这种方法允许第三方开发人员在不访问源代码的情况下向您的应用程序添加功能。 您还可以将多个平台的插件捆绑在一起，并让CFPlugIn在运行时透明地加载适当的插件。 您可以使用CFPlugIn向您的应用程序添加插件功能或为其编写插件。


### CFPlugInInstance

已废弃


### CFPropertyList

CFPropertyList提供了一些功能，可以将属性列表对象与多种序列化格式（例如XML）进行相互转换。 表示CFPropertyList对象的CFPropertyListRef类型是属性列表对象的抽象类型。 根据用于创建属性列表的XML数据的内容，CFPropertyListRef可以是任何属性列表对象：CFData，CFString，CFArray，CFDictionary，CFDate，CFBoolean和CFNumber。 请注意，如果使用属性列表生成XML，则属性列表中任何词典的键都必须是CFString对象。


### CFReadStream
CFReadStream提供了一个用于同步或异步读取字节流的接口。 您可以创建从内存块，文件或通用套接字读取字节的流。 读取之前，需要使用CFReadStreamOpen打开所有流。
使用CFWriteStream写入字节流。 CFNetwork框架定义了一种其他类型的流，用于读取对HTTP请求的响应。


### CFRunLoop
### CFRunLoopObserver
### CFRunLoopSource
### CFRunLoopTimer
### CFSet

### CFSocket

CFSocket是使用BSD套接字实现的通信通道。

### CFString

### CFStringTokenizer

CFStringTokenizer允许您以与语言无关的方式将字符串标记为单词，句子或段落。 它支持诸如日语和中文之类的语言，这些语言不会用空格来分隔单词，还可以分解德语化合物。 您可以获取令牌的拉丁语转录。 它还提供了语言识别API。


### CFTimeZone

### CFTree
您可以使用CFTree创建表示信息的层次结构的树结构。 在这样的结构中，每个树节点都只有一个父树（根树除外，后者没有父树），并且可以有多个子树。 结构中的每个CFTree对象都有一个与之关联的上下文。 此上下文包括一些程序定义的数据以及对该数据进行操作的回调。 程序定义的数据通常用作确定CFTree对象在结构中适合放置的位置的基础。 所有CFTree对象都是可变的。


### CFType

所有其他Core Foundation不透明类型均源自CFType。 为CFType定义的函数，回调，数据类型和常量可由任何派生的不透明类型使用。 因此，CFType函数被称为“多态函数”。您可以使用CFType函数来保留和释放对象，比较和检查对象，获取对象和不透明类型的描述以及获取对象分配器。

### CFURL

### CFUserNotification

CFUserNotification对象在屏幕上显示一个简单的对话框，并且可以选择接收用户的反馈。 对话框的内容可以包括标题，消息，图标，文本字段，弹出按钮，单选按钮或复选框，以及最多三个普通按钮。 在没有用户界面，但有时需要与用户交互的过程中，请使用CFUserNotification。

### CFUUID

### CFWriteStream

CFWriteStream提供了一个用于同步或异步写入字节流的接口。 您可以创建将字节写入内存块，文件或通用套接字的流。 写入之前，需要使用CFWriteStreamOpen打开所有流。


### CFXMLNode
### CFXMLParser
### CFXMLTree



