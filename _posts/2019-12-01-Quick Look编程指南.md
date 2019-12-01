---
layout:     post
title:      "Quick Look编程指南"
subtitle:   "持续更新中..."
date:       2019-11-25 22:38:02
author:     "CoderLeonidas"
catalog: true
tags:

- Cocoa
- macOS
- 图形
- 翻译


---



# Quick Look编程指南

# 0 介绍
QuickLook 是一项在OSX 10.5中引进的技术，可以使得客户端程序如Spotlight、Finder等来展示文档的缩略图和全尺寸预览。常见类型的文档，比如HTML, RTF, plain text, TIFF, PNG, JPEG, PDF, DAE, and QuickTime movies，是自动支持QL的。但是，具有不常见甚至私有内容类型的文档的应用程序仍可以使用QL功能。这些应用程序可以包涵Quick Look生成器：将给定文档的本机格式转换为Quick Look可以显示给用户的格式的插件。

本文档描述了QL技术以及解释了作为app开发者的你如何去创建一个生成器来让QL展示你的文档的缩略图或者预览图片。尽管QL生成器设计成了CFPlugIn风格
的bundles，但为你处理好了插件实现的的所有详细信息。同时，尽管QL生成器的编程接口是ANSI C的接口，你仍然可以使用OC代码调用Cocoa框架的方法来写生成器。


## 本文档组织结构：

QL编程指南有以下几个章节：

[Quick Look and the User Experience]() 描述了QL技术的作用以及指出了app使用这项技术的好处。它也定义了在QL中有着特殊含义的一些术语。

[Quick Look Architecture ]() 描述了QL的几个组件，包括他们各自的角色以及相互间如何通信

[Creating and Configuring a Quick Look Project]() 解释了如何创建一个QL生成器工程以及如何指定生成器的属性

[Overview of Generator Implementation ]() 总结了生成缩略图和预览图片的方法，并确定了每种方法的最佳上下文(最佳使用场景？？)

[Drawing Thumbnails and Previews In a Graphics Context ]() 显示了如何在针对位图，单页矢量和多页矢量图形优化的图形上下文中绘制缩略图和预览

[Dynamically Generating Previews ]() 讨论了如何在受支持的内容类型（如RTF或HTML）中动态生成基于文本的预览； 对于HTML预览，它还显示了如何包括图像之类的附件。

[Saving Previews and Thumbnails in the Document]() 描述了app保存文档中缩略图或者预览图片的路径、生成器为QL检索图片的路径。同时它也描述了当图片数据以Image I/O框架支持的格式返回给QL时要使用的功能(函数)

[Assigning Core Graphics Images to Thumbnails ]() 展示了当一个图片格式不为Image I/O框架所支持，如何将其返回(比如一个CGImage)

[Canceling Previews and Thumbnails]() 解释了如何取消由QL发起请求的缩略图或预览的生成过程

[Debugging and Testing a Generator]() 描述了用于调试和测试QL生成器的工具和技术

## 更多请参阅：

请查阅以下文档，以获取Quick Look生成器功能和常量的描述：


[QLPreviewRequest Reference]()
[QLThumbnailRequest Reference]()

由于生成一个缩略图或者预览图片通常需要绘制或创建一个图片对象，所以以下文档可能也用得上：

[Quartz 2D Programming Guide]()
[mage I/O Reference Collection]()
[Cocoa Drawing Guide]()
[Core Image Programming Guide]()


# 1 Quick Look and the User Experience - QL与用户体验

OS X系统上的某些应用程序为用户提供了文档文件列表。这些应用程序包括Finder，Spotlight和Time Machine。这些应用程序显示文档图标，文件名以及与该文档相关的元数据，但是通常此信息不足以使用户将一个文档与另一个文档区分开。OS X版本 10.5之前，要想通过内容来识别特定文档，用户必须打开列表中的每个文档（通常要求他们启动应用程序），直到找到所需的文档为止。 不用说，这是一个耗时的过程。

QL是OS X 10.5版中引入的一项功能，使用户可以快速发现列出的文档的内容，无论是缩略图还是全尺寸预览图像，而无需启动文档的应用程序。 以下各节介绍了“QL"功能，并确定了那些可能会为其文档生成“QL”缩略图和预览的应用程序。

## 缩略图和预览

快速查看显示两种文档表示形式：缩略图和预览。 这两种表示满足不同的需求。

缩略图是描述文档的静态图像。 尽管大小可能有所不同，但通常小于预览，并且大于文档图标。 缩略图的目的是为用户提供在相当小的范围内的文档内容概念。 在较大尺寸下，某些文档（例如图像文件）的缩略图可能与全尺寸预览一样有用。 但是在较小的尺寸上，缩略图在传达文档中包含的内容时可能不会比文档图标更好。

预览是文档的较大表示形式，通常是文档的全尺寸呈现，它包含在“快速查看”面板中（请参见图1-2）。 快速查找显示预览，而无需在其自己的应用程序中打开文档。 当缩略图不可用或显示的细节不足以使他们将一个文档与另一个文档区分开时，用户请求预览。

客户端可以一次显示多个缩略图，而预览通常一次显示一个。




## QL操作

要了解Quick Look对用户体验的贡献，请考虑如何使用它。 如果您在Spotlight中搜索一个项目（例如“ elephant”），然后单击“显示全部”选项，则可能会看到一个Finder窗口，如图1-1所示。


Figure 1-1  Thumbnails in the Finder’s Cover Flow view
![](https://i.loli.net/2019/11/26/uUNioWnhTLEOypD.png)

Finder的Cover Flow视图中间的图像是Quick Look生成的缩略图。 缩略图出现在OS X v10.5中的不同位置。 除了Cover Flow之外，当需要多个预览时，它们还会以图标模式和列模式显示在Finder中，以图标堆栈的形式出现在Dock中，并出现在快速查看索引表中。 当“快速查看”无法生成预览（但可以生成缩略图）时，“快速查看”面板中也会显示缩略图。

如果用户想仔细查看一个文档，可以在Spotlight或Finder窗口中选择它，然后按空格键。 快速查看将显示文档的全尺寸预览图像，类似于图1-2中的图像。 （在Finder中，您也可以按Command-Y来查看所选项目的预览。）

Figure 1-2  A Quick Look preview of an image

![](https://i.loli.net/2019/11/26/HBx7a4lgZzKf1nd.png)


快速查看预览不仅包括静态图像和文档，而且还可以包括QuickTime电影，如图1-3中的示例所示。

Figure 1-3  A Quick Look preview of a movie

![](https://i.loli.net/2019/11/26/opOTyVRfGx5hEJi.png)

当用户在Finder中请求多个文档的预览时，“快速查看”面板使他们可以循环浏览这些预览或一次在索引表中查看它们（作为缩略图）。 快速查看面板还包含一些控件，这些控件允许用户动态调整预览的大小，扩展预览以占据屏幕，关闭面板，并（取决于文档类型）执行“播放电影”和“添加到iPhoto”之类的操作。

## QL开发
在结构上，Quick Look具有使用者（或客户端）一侧，以及将缩略图和预览图像提供给使用者一侧以进行显示的一侧。 （要详细了解Quick Look的各个部分以及它们如何协同工作，请参阅Quick Look体系结构。）Quick Look的客户端分别请求列出和所选文档的缩略图和预览，并接收要显示的图像。


如果格式是其本机类型之一，则“快速查找”支持显示文档缩略图和预览。 本地快速查找类型是纯文本，RTF，HTML，PDF，图像（各种标准格式，例如JPG，PNG和TIFF）以及QuickTime影片和声音。 但是，如果文档不是本机类型之一，则文档的应用程序如果要利用“快速查找”功能，则必须包括“快速查找”生成器。 生成器是一个bundle
，用于以一种本机类型来创建应用程序文档的表示形式，以显示为预览和缩略图。 快速查找生成器的目的是根据需要并尽可能有效地提供一种本地快速查找类型(native Quick Look types )的文档的缩略图或预览图像。


# 2 Quick Look Architecture 

以下各节检查快速外观的体系结构。 此体系结构的总体情况可帮助您了解生成器的作用和约束。


## QL 的生产者消费者模型


快速查看的体系结构基于消费者-生产者模型。 使用者（或客户端）是想要显示缩略图和预览文档表示的应用程序。 体系结构的生产方将这些表示提供给消费者。 （某些Quick Look客户端是Finder，Spotlight和FileSync之类的系统应用程序。 客户端有权限访问公共函数`QLThumbnailImageCreate`，以及`QLPreviewPanel Class Reference`中描述的QL预览面板。图2-1说明了Quick Look的体系结构。

Figure 2-1  The Quick Look architecture

![](https://i.loli.net/2019/11/26/lqBANjXHzxOQCTd.png)


QL的消费者有3个组件：document reader(由自定义视图和面板组成)、相应的display bundles、以及一个用于实现与客户端通信的SPI。这些组件中的每一个都在支持消费者方面扮演着特定的角色：

- Document reader：快速查看实现了为显示文档预览而定制的视图（NSView）和面板（NSPanel）。 除了预览内容之外，该视图还可能包括（由客户选择）用于操纵预览的控件，例如，页面前进，页面后退，开始播放，快退和文本搜索。 客户端应用程序可以选择将该视图嵌入其用户界面中。 “快速查看”面板包含一个“快速查看”视图和各种控件，这些控件使用户可以对预览进行一些操作，例如使预览图像全屏显示或开始幻灯片播放。

- Display bundles：快速查看视图本身不显示文档预览，而是用于显示bundle的委托。 每种本机文档类型都有一个“快速查看”显示bundle。 有关快速本机类型的讨论，请参见`《 Developing for Quick Look》`。 如果文档不是本机类型，则必须将其转换为一个文档才能显示为“快速查看”缩略图或预览。

- Consumer SPI：客户端应用程序通过该界面与Quick Look进行对话，请求预览和缩略图，并访问Quick Look文档阅读器

Quick Look的“生产者”部分基于插件体系结构，如果这些文档不是本机Quick Look类型之一，则使应用程序能够提供其文档的缩略图和预览。 Quick Look的使用者和生产者部分通过一个或多个Mach端口进行通信。
 

## 进一步研究QL的守护进程和生成器

快速查看的“生产者”端是进行第三方开发的地方，因此值得仔细观察。 它由一个或多个Quick Look守护程序和多个Quick Look生成器组成。 图2-2显示了这些事物如何相互关联。

Figure 2-2  Quick Look provider component

![](https://i.loli.net/2019/11/26/XEyqHWC9r7hGNQZ.png)

快速查看生成器是基于CFPlugIn的bundle，可提供缩略图图像和应用程序文档的预览。生成器的工作是针对接收到的每个预览或缩略图请求，将文档数据转换为Quick Look本机类型之一。守护程序在首次接收到对文档预览或缩略图的请求时将加载生成器。它使用文档的内容类型UTI来定位与特定文档关联的生成器，该内容类型在生成器的信息属性列表(plist)中指定。它在应用程序包内或在生成器的标准文件系统位置之一（例如`/Library/QuickLook`）中查找生成器。Quick Look生成器必须是64位二进制文件，如果需要支持较早的系统，则必须是通用二进制文件。

Quick Look守护程序（快速查找）是一个匿名的后台应用程序，它充当基于CFPlugIn的生成器的主机(host)。它通过Mach端口与Quick Look的使用者端进行通信，并且（如上所述）在第一次接收到其格式不是本机类型之一的文档的预览或缩略图请求时定位并加载生成器。 它将客户的请求传达给生成器并返回他们的响应。

使用守护程序作为使用者SPI和生成器之间的中介是有优势的。如果“快速查看”守护程序崩溃，则可以立即重新启动它以从上次中断的地方恢复服务。如果生成器不是线程安全的或出于任何其他原因需要隔离，则可以运行单独的守护程序来处理对该生成器的请求。 当守护程序空闲时，Quick Look可以终止它，从而释放系统资源。

在所有架构部分都准备就绪之后，我们来看看当Finder等客户端应用程序要求显示文档预览时会发生什么。用户打开一个文件夹，显示各种类型的文档列表。 这些文档中的一些是本机“快速查看”类型的，而另一些则特定于某些应用程序。 用户选择一个文档（例如JPG文件），然后从“文件”菜单中选择“快速查看预览”命令。 在快速查看中，将按以下顺序进行操作：

1. 客户端（Finder）将一条消息发送到Quick Look的使用者部分，请求预览文档。
2. 快速查看发现文档格式是本机类型，因此它将加载适当的显示bundle（如有必要）
3. 该显示bundle在document reader （即“快速查看”视图中，即“快速查看”面板的内容视图）中绘制文档。

接下来，用户请求预览非“快速查找”本机类型的文档。 发生以下顺序的步骤：


1. 客户端（Finder）向Quick Look的使用者部分发送一条消息，请求预览文档。
2. Quick Look框架发现文档格式不是本机类型，因此将消息转发到Quick Look守护程序
3. 守护程序使用文档的内容类型UTI，找到合适的生成器并在必要时加载它。
4. 它将预览请求转发给生成器，生成器创建预览并返回预览或告诉生成器在哪里找到它。
5. 守护程序将生成器的响应返回给Quick Look的使用者部分。
6. “快速查找”将加载适当的显示bundle（如有必要）。
7. 展示bundle将在文档阅读器中绘制文档。

## 安装QL生成器

您可以将Quick Look生成器存储在应用程序bundle中（在`MyApp.app/Contents/Library/QuickLook/`中）或标准文件系统位置之一中：

- `~/Library/QuickLook` --第三方生成器，仅可用于登录使用

- `/Library/QuickLook` -- 第三方生成器，系统的所有用户均可访问

![](https://i.loli.net/2019/11/26/4OIlGjvYWp7HiNq.png)

- `/System/Library/QuickLook` -- 苹果提供的生成器，系统的所有用户均可使用

![](https://i.loli.net/2019/11/26/CEBUTyJ3uPIdXSH.png)


当“快速查找”搜索要使用的生成器时，它首先在相关应用程序的bundle中查找它，然后按照上面列表中给出的顺序在标准文件系统位置中查找它。如果两个生成器具有相同的UTI，则“快速查找”将使用在此搜索顺序中找到的第一个生成器。如果两个生成器在同一级别上声明了相同的UTI（例如，在`/Library/QuickLook`中），则无法确定将选择其中哪一个。


# 3 Creating and Configuring a Quick Look Project

Quick Look生成器的Xcode项目源自特殊的模板，该模板设置了项目的重要方面。 但是，您仍然必须指定特定于生成器的配置信息并为生成器添加任何资源，通常在编写任何代码之前。

## 创建和设置项目

要创建Quick Look生成器项目，请先从Xcode应用程序的“文件”菜单中选择“新建项目”。 在项目创建助手中，在项目模板列表中选择“ Quick Look插件”（如图3-1所示），然后单击“下一步”。

Figure 3-1  Choosing the Quick Look plug-in template

![](https://i.loli.net/2019/11/26/wB4pQbDlsv2YiGk.png)

为项目指定名称和位置后，Xcode将显示一个项目窗口，类似于图3-2中的示例。

Figure 3-2  Default items in a Quick Look plug-in project

![](https://i.loli.net/2019/11/26/CnIavEPRBiGUNw6.png)


此窗口中的以下项目与“快速查看”有一些特殊的关联：

- main.c - 该文件包含基于CFPlugin的插件所需的所有代码。 您不必添加或修改任何此代码。
- GenerateThumbnailForURL.c 和 GeneratePreviewForURL.c - 第一个文件包含用于代码回调`GeneratePreviewForURL`和`CancelPreviewGeneration`的代码模板； 第二个文件包含用于回调`GenerateThumbnailForURL`和`CancelThumbnailGeneration`的代码模板。


如果您的实现代码将是`Objective-C`，请确保在Xcode中将这些文件的扩展名从`c`更改为`m`（即，通过选择文件并从File菜单中选择Rename）。


尽管“快速外观”生成器没有（也不应）将`nib`文件作为资源，但是您可以根据需要添加其他资源。

## 项目配置

Quick Look生成器必须是64位二进制文件，如果需要支持较早的系统，则必须是通用二进制文件。

快速查看生成器项目的信息属性列表（`Info.plist`）包括一些特殊属性，除了标准属性（例如`CFBundleIdentifier`和`CFBundleVersion`）外，还应设置其值。 以下各节描述了这些属性。

Listing 3-1  The subproperties of CFBundleDocumentTypes

```xml
    <key>CFBundleDocumentTypes</key>
    <array>
        <dict>
            <key>CFBundleTypeRole</key>
            <string>QLGenerator</string>
            <key>LSItemContentTypes</key>
            <array>
                <string>SUPPORTED_UTI_TYPE</string> // change this!
            </array>
        </dict>
    </array>
```
![](https://i.loli.net/2019/11/26/k8Az3Bfq9doTcGV.png)


将字符串`SUPPORTED_UTI_TYPE`替换为一个或多个UTI，以标识此生成器为其生成缩略图和预览的文档的内容类型。 例如，[QuickLookSketch](https://developer.apple.com/library/archive/samplecode/QuickLookSketch/Introduction/Intro.html#//apple_ref/doc/uid/DTS10004031)示例项目为Sketch文档指定UTI：

```xml
<key>CFBundleDocumentTypes</key>
    <array>
        <dict>
            <key>CFBundleTypeRole</key>
            <string>QLGenerator</string>
            <key>LSItemContentTypes</key>
            <array>
                <string>com.apple.sketch2</string>
                <string>com.apple.sketch1</string>
            </array>
        </dict>
    </array>
```
有关文档内容类型的统一类型标识符（UTI）的更多信息，请参见[Uniform Type Identifiers Overview](https://developer.apple.com/library/archive/documentation/FileManagement/Conceptual/understanding_utis/understand_utis_intro/understand_utis_intro.html#//apple_ref/doc/uid/TP40001319)。

如清单3-2所示，快速查看生成器项目中`Info.plist`的很大一部分是与`CFPlugIn`相关的属性。 您不必编辑这些属性。

Listing 3-2  CFPlugIn properties

```xml
    <key>CFPlugInDynamicRegisterFunction</key>
    <string></string>
    <key>CFPlugInDynamicRegistration</key>
    <string>NO</string>
    <key>CFPlugInFactories</key>
    <dict>
        <key>27EB40F9-21D6-4438-9395-692B52DB53FB</key>
        <string>QuickLookGeneratorPluginFactory</string>
    </dict>
    <key>CFPlugInTypes</key>
    <dict>
        <key>5E2D9680-5022-40FA-B806-43349622E5B9</key>
        <array>
            <string>27EB40F9-21D6-4438-9395-692B52DB53FB</string>
        </array>
    </dict>
    <key>CFPlugInUnloadFunction</key>
    <string></string>
```

## 其他属性列表键

您可以在“快速查找”生成器的信息属性列表（Info.plist）中指定这些其他键/值对：

 Key | Allowed value | Description
------------- | ------------- | -------------
QLThumbnailMinimumSize  | Real number \<real>n\</real> | 指定生成器沿缩略图的一维（以点为单位）的最小使用大小。 对于小于此值的缩略图，“快速查找”不会调用GenerateThumbnailForURL回调函数。 默认大小为17。如果生成器足够快，则可以删除此属性，以便缩略图图像可以出现在标准列表中。
QLPreviewWidth  | Real number \<real>n\</real> | 此数字为“快速查找”提示了预览的宽度（以点为单位）。 如果生成器花费太长时间来生成预览，它将使用这些值。
QLPreviewHeight  | Real number \<real>n\</real> | 此数字为“快速查找”提示了预览的高度（以控制生成器是否可以在主线程以外的线程中运行为单位）。 如果生成器花费太长时间来生成预览，它将使用这些值。
QLSupportsConcurrentRequests  | YES or NO | 控制生成器是否可以处理并发的缩略图和预览请求。
QLNeedsToBeRunInMainThread  | YES or NO | 控制生成器是否可以在主线程以外的线程中运行


属性`QLSupportsConcurrentRequests`和`QLNeedsToBeRunInMainThread`是影响生成器的多线程特性的快速查看属性。 它们在`生成器和线程安全`中讨论。

# 4 Overview of Generator Implementation 


快速查看生成器API为您提供了几种实现生成器的方法。 本章介绍了它们的含义，并根据其文档类型提出了最适合应用程序的方法。 它还讨论了与Quick Look生成器有关的线程安全和多线程问题。

本章仅总结了缩略图和预览的生成。 有关如何取消缩略图和预览的生成的讨论，请参见`Canceling Previews and Thumbnails`。

## QL生成器实现API

快速查找框架中的头文件`QLGenerator.h`声明了快速查找生成器的编程接口。 （另一个头文件`QLBase.h`也位于`Headers`文件夹中，但是此文件仅包含Quick Look公共和私有接口使用的各种宏的定义。）生成器的编程接口在缩略图请求和预览请求之间划分。 分别由不透明类型`QLThumbnailRequestRef`和`QLPreviewRequestRef`表示。 API分为三类：

- Callbacks回调 ：

> 生成器必须实现类型为`GenerateThumbnailForURL`的回调函数，才能创建和返回文档的缩略图表示。 他们必须实现类型为`GeneratePreviewForURL`的回调函数，才能创建和返回文档预览。 如`Creating and Configuring a Quick Look Project,`中所述，生成器的Xcode模板使回调函数的默认名称与其类型名称相同。
>
>  可以实现一对附加的回调函数，以取消生成器当前正在执行的预览和缩略图的生成。 有关这些回调的更多信息，请参见取消预览和缩略图。

- 用于生成缩略图和预览的函数

> 快速查看为生成器提供了一系列功能选择，以创建和返回缩略图和预览。 例如，`QLThumbnailRequestCreateContext`和`QLPreviewRequestCreateContext`函数提供了用于绘制位图和基于矢量的图像的图形上下文。您可以使用`QLPreviewRequestSetDataRepresentation`函数返回嵌入式或动态生成的预览，通常用于带有附件的HTML内容。 使用`QLThumbnailRequestSetImage`函数，您可以返回表示文档的静态缩略图。

> `Approaches to Thumbnail and Preview Generation `更详细地描述了这些功能和相关功能，并确定了最适合其使用的情况。

- 返回有关请求或生成器信息的函数

> `QLGenerator.h`中的其余功能允许您获取预览或缩略图请求的属性，或获取与它们有关的其他数据。 例如，`QLThumbnailRequestCopyURL`函数返回标识请求为其提供缩略图的文档的URL。 `QLThumbnailRequestGetGeneratorBundle`函数返回对生成器包的引用（`CFBundleRef`）。 而且`QLPreviewRequestCopyContentUTI`函数返回当前文档内容的UTI标识符（例如`com.apple.sketch1`）。

---

>  注意： 有个重要的区别在生成器编程中需要时刻记住，那就是`options`和`properties`的区别。两者都是Quick Look函数中CFDictionaryRef参数的名称。但是回调函数`GenerateThumbnailForURL`和`GeneratePreviewForURL`中的`options`参数是从客户端到生成器的选项或提示字典，用于说明如何处理请求。`properties `参数是用于创建缩略图和预览的`QLThumbnailRequest`和`QLPreviewRequest`函数中的最后一个参数。 `properties `字典包含补充创建的缩略图或预览的数据。

## 缩略图和预览生成的方法
生成缩略图和预览的方法以及使用的“快速查看”功能取决于生成器所要使用的文档类型。 问问自己有关文档的以下问题：

- 它是bundled的（例如，Pages文档）还是非bundled的（或平面的）？
- 它是否包含图形或文本？ 还是图形和文字？
- 如果是图形，是位图还是矢量图像？
- 它是一页还是多页？


## 生成器与线程安全


出于性能原因，Quick Look守护程序（`quicklookd`）更喜欢在自己的线程中运行生成器，通常与其他生成器并发运行，或者当该生成器正在处理多个文档时甚至与同一生成器并发运行。 鉴于此，当您为生成器编写代码时，会出现一些线程安全性问题：

- 生成器代码本身是线程安全的吗？
- 生成器在当前上下文中调用的框架是否是线程安全的？
有关系统的哪些部分是线程安全的讨论，请参见[线程安全摘要](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/Multithreading/ThreadSafetySummary/ThreadSafetySummary.html#//apple_ref/doc/uid/10000057i-CH12)。



# 5 Drawing Thumbnails and Previews In a Graphics Context

# 6 Dynamically Generating Previews

# 7 Saving Previews and Thumbnails in the Document

# 8 Assigning Core Graphics Images to Thumbnails 

# 9 Canceling Previews and Thumbnails 

# 10 Debugging and Testing a Generator


 