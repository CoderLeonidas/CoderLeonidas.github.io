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
- 它是否包含图形或文本？ 还是图形和文本都有？
- 如果是图形，是位图还是矢量图像？
- 它是一页还是多页？

当然，是要缩略图还是预览取决于您选择的方法。如果请求的缩略图尺寸不大于常规文档图标，则该尺寸的缩略图可能不会比该图标好。如果请求是预览多页文档，那么您仅显示文档的首页还是全部？论请求是缩略图还是预览，生成器的性能都至关重要。例如，当客户请求缩略图时，它可以要求他们提供许多不同的文档；效率低下的生成器会使客户端的缩略图显示缓慢。如果客户要求预览200页以上的文档，则可能只应包含足够的文档供用户识别。对于生成器，您应该采用适当的内存管理做法和适当的多线程策略。有关多线程生成器和线程安全性问题的更多信息，请参见《生成器与线程安全》。


如果要为bundled的文档指定静态缩略图和预览图像，则可以采用最简单的方法-甚至不需要生成器。只需让您的应用程序将图像放置在名为QuickLook的子文件夹中的文档包中即可；缩略图的图像文件应命名为`Thumbnail.ext`，预览文件的名称应命名为`Preview.ext`（其中ext是扩展名，例如tiff，png或jpg）。如果决定采用这种方法，则不应创建生成器。


以编程方式，根据文档和其他情况，您可以采用以下方法之一生成缩略图和预览：

- 如果文档是包含位图图形，矢量图形甚至文本的单页（通常是预览的图形元素），您可以在分别由`QLThumbnailRequestCreateContext`或`QLPreviewRequestCreateContext`函数返回的图形上下文中绘制缩略图或预览。
- 如果文档具有多个页面的矢量图形或文本，则可以在`QLPreviewRequestCreatePDFContext`提供的图形上下文中以`PDF`内容形式绘制预览。您可以调用常规的`Core Graphics`函数来绘制预览图像
- 对于任何类型的文档，应用程序都可以将缩略图和预览图像作为文档数据的一部分写入，生成器分别使用函数`QLThumbnailRequestSetImageWithData`和`QLPreviewRequestSetDataRepresentation`检索并返回该缩略图和预览图像。图4-1说明了这种方法。对于预览，必须通过`contentTypeUTI`参数指定预览数据所在的本机“快速查找”类型。对于缩略图，返回的数据必须采用可由`Image I/O`框架处理的格式：JPG，TIFF，PNG等。

<center>Figure 4-1  Returning preview data stored in the document</center>
<center>![](https://i.loli.net/2019/12/03/6zgiLp1WETUABPc.png)</center>
 
- 对于多页文档，通常是文本文档，生成器可以“即时”动态生成预览，并使用`QLPreviewRequestSetDataRepresentation`函数将其返回。功能和参数的这种组合告诉Quick Look使用`WebKit`处理预览的布局。 在函数的最后一个参数，即属性字典中，您可以在HTML中指定附件（例如图像，声音，甚至是通讯簿卡之类的东西）。为了使这种方法可行，文档数据必须可以转换为HTML。
- 当您无法（通过`QLThumbnailRequestSetImageWithData`）提供快速查看格式适用于`Image I/O`框架缩略图版本时，但是您可以生成其他格式的序列化缩略图，则可以使用`QLThumbnailRequestSetImage`函数来 将此图像返回到快速查看。



## 生成器与线程安全


出于性能原因，Quick Look守护程序（`quicklookd`）更喜欢在自己的线程中运行生成器，通常与其他生成器并发运行，或者当该生成器正在处理多个文档时甚至与同一生成器并发运行。 鉴于此，当您为生成器编写代码时，会出现一些线程安全性问题：

- 生成器代码本身是线程安全的吗？
- 生成器在当前上下文中调用的框架是否是线程安全的？
有关系统的哪些部分是线程安全的讨论，请参见[线程安全摘要](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/Multithreading/ThreadSafetySummary/ThreadSafetySummary.html#//apple_ref/doc/uid/10000057i-CH12)。
- 生成器调用的生成器代码或框架代码是否可以在非主线程中运行？

如果您可以确定这些问题的答案，则可以通过在生成器的信息属性列表（Info.plist）中设置`QLSupportsConcurrentRequests`和`QLNeedsToBeRunInMainThread`属性来配置生成器以实现最佳性能。（如果您不确定上述任何问题的答案，请考虑线程安全性方面最保守的答案。）表4-1总结了当您为这两个属性分配不同的值时，Quick Look假定的线程安全状态。

<center>Table 4-1  Quick Look properties for specifying the thread-safety status of the generator</center>

<center>![](https://i.loli.net/2019/12/03/KeG2IPA5rxnphjT.png)</center>

有关线程安全问题（包括Carbon和Cocoa框架的线程安全状态）的信息，请参阅《[线程编程指南](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/Multithreading/Introduction/Introduction.html#//apple_ref/doc/uid/10000057i)》。

# 5 Drawing Thumbnails and Previews In a Graphics Context

对于主要或仅包含图形和图像的文档的预览或缩略图，生成器可以采用的最佳方法是在Quick Look提供的图形上下文中绘制该文档的图像。 生成器直接在客户端中绘制文档图像-图形上下文充当客户端应用程序表面上的一种窗口。 这样，您可以避免创建映像并将其压缩为本机类型，然后要求客户端对其解压缩并加载的开销。 

 提供了三种图形上下文，每种上下文分别用于不同类型的文档：
 
- 用于绘制一张或多张适合单页面的位图图像的图形上下文
- 用于绘制一张或多张适合单页面的矢量图像的图形上下文
- 用于绘制多页矢量图像的图形上下文

《在图形上下文绘制文档图片》描述了如何在前两种情况下，绘制缩略图或者预览。
《 在PDF上下文中绘制预览》讨论了第三种图形上下文，并解释了它的用法。

## 在图形上下文绘制文档图片

在图形上下文中绘制单页预览和缩略图的策略是相同的。实现适当的回调函数`GenerateThumbnailForURL`或`GeneratePreviewForURL`，以将给定文档（由`CFURLRef`参数定位）读入内存。然后，通过调用`QLThumbnailRequestCreateContext`或`QLPreviewRequestCreateContext`获得Quick Look图形上下文，并在提供的上下文中绘制缩略图或预览图像。清单5-1显示了用于生成Sketch文档预览的代码。

<center>Listing 5-1  Drawing a Sketch preview in a Quick Look graphics context</center>

```c++
OSStatus GeneratePreviewForURL(void *thisInterface, QLPreviewRequestRef preview, CFURLRef url, CFStringRef contentTypeUTI, CFDictionaryRef options)
{
    @autoreleasepool{
        // Create and read the document file
        SKTDrawDocument* document = [[SKTDrawDocument alloc] init];
 
        if (![document readFromURL:(NSURL *)url ofType:(NSString *)contentTypeUTI]) {
            return noErr;
        }
 
        NSSize canvasSize = [document canvasSize];
 
        // Preview will be drawn in a vectorized context
        CGContextRef cgContext = QLPreviewRequestCreateContext(preview, *(CGSize *)&canvasSize, false, NULL);
        if(cgContext) {
            NSGraphicsContext* context = [NSGraphicsContext graphicsContextWithGraphicsPort:(void *)cgContext flipped:YES];
            if(context) {
                [document drawDocumentInContext:context];
            }
            QLPreviewRequestFlushContext(preview, cgContext);
            CFRelease(cgContext);
        }
    }
    return noErr;
}
```



在提供的图形上下文中绘制预览或缩略图之前，请确保保存当前上下文，然后在完成绘制后还原该上下文。 然后，应使用`QLPreviewRequestFlushContext`或`QLThumbnailRequestFlushContext`刷新上下文，然后如上例所示释放它.`QLPreviewRequestCreateContext`和`QLThumbnailRequestCreateContext`函数具有相同的参数集。 第一个参数标识传递到回调中的预览请求或缩略图请求对象。 另一个参数对创建的图形上下文有更直接的影响。

- 第二个参数（函数声明中名为size的参数）是要绘制的图像大小，以像素或点为单位，分别取决于图形上下文是位图还是矢量。
- 第三个参数（isBitmap）告诉“快速查看”，返回的图形上下文是否适合位图或矢量图形。在上面的示例中，请求的矢量优化图形上下文带有错误值。
- 第四个也是最后一个参数是属性字典，您可以将其传递回Quick Look，作为处理绘制图像的提示。有关详细信息，请参见`QLPreviewRequest`参考和`QLThumbnailRequest`参考。


生成GenerateThumbnailForURL回调的生成器可以在选项目录中传递一个浮点值，该浮点值指定“快速查找”缩放缩略图图像的程度。（您可以使用`kQLThumbnailOptionScaleFactorKey`键访问此值。）。如果您使用从`QLThumbnailRequestCreateContext`返回的图形上下文为缩略图绘制矢量图像，则不必担心缩放图像。 只需以给定尺寸（以磅为单位）正常绘制即可。快速查找会创建一个具有指定大小乘以比例因子（以像素为单位）的上下文，而且还会应用适当的仿射变换，以便绘图上下文在生成器中显示为指定大小。


## 在PDF上下文中绘制预览

如果您的应用程序具有（可能）具有一页以上的矢量图形的文档，则应考虑使用`QLPreviewRequestCreatePDFContext`函数创建用于绘制预览的图形上下文。此函数返回适合PDF内容的图形上下文。 该过程类似于在“图形上下文中绘制文档图像”中描述的过程。 但是，有一些重要的区别：

- `QLPreviewRequestCreatePDFContext`的某些参数与`QLPreviewRequestCreateContext`的参数不同：
	- 第二个参数（mediaBox）是指向矩形的指针，该矩形定义了PDF页面的位置和大小。
	- 第三个参数（auxiliaryInfo）是包含辅助PDF信息的字典。
- 您必须在绘制每个页面之前通过调用`CGPDFContextBeginPage`并在完成绘制页面后调用`CGPDFContextEndPage`。

与`QLPreviewRequestCreateContext`一样，完成预览图的绘制后，请确保调用`QLPreviewRequestFlushContext`。

# 6 Dynamically Generating Previews

如果您可以轻松地将文档从其原始格式转换为适当的“快速查看”格式，则生成器可以执行该转换以用于该文档的预览。这种转换对于文本快速查看格式（HTML，RTF和纯文本）最有用，但是您也可以生成快速查看支持的任何格式的预览。 例如，如果您将代表3D模型的文档转换为数字资产交换（DAE）格式，则“快速查找”可以显示预览界面，允许缩放和旋转模型。（请注意，图形上下文仍然是向“快速外观”提供二维图像数据的最有效方法；有关详细信息，请参见在《图形上下文中绘制缩略图和预览》。）

动态创建文本表示形式讨论了一种动态创建文本预览（在本例中为RTF）的方法。

如果可以为预览生成HTML数据，则还可以包括图像，CSS样式表和JavaScript脚本等项目的附件。 您可以使用这些项目在“快速查看”预览界面中提供详细的布局和交互性。 《生成丰富的HTML》描述了这种方法。

## 动态创建文本表示

对于包含非人类可读格式的文本信息的文档类型，动态预览生成可能很有用。此类文档类型的一个示例是Xcode使用的本地化字符串格式（有关更多信息，请参见本地化字符串资源）。本地化的字符串文档可以以二进制或XML格式存储，因此Quick Look生成器可以解释这些格式并生成RTF预览，该预览使用字体和颜色使文档内容易于阅读。清单6-1中的代码示例说明了创建此类预览的一种方法。

`QLPreviewRequestSetDataRepresentation`函数调用是此代码中最重要的部分。 生成器为此功能提供了两个参数：用于预览的RTF数据和用于指示数据对象包含哪个本机快速外观的UTI常数。

<center>Listing 6-1  Generating a preview in RTF format</center>

```objc
OSStatus GeneratePreviewForURL(void *thisInterface, QLPreviewRequestRef preview, CFURLRef url, CFStringRef contentTypeUTI, CFDictionaryRef options)
{
    @autoreleasepool {
 
        // Load the strings dictionary from the URL
        NSDictionary *localizationDict = [NSDictionary dictionaryWithContentsOfURL:(__bridge NSURL *)url];
        if (!localizationDict) return noErr;
 
        // The above might have taken some time, so before proceeding make sure the user didn't cancel the request
        if (QLPreviewRequestIsCancelled(preview)) return noErr;
 
        // Set up for producing attributed string output
        NSDictionary *keyAttrs = @{ NSFontAttributeName : [NSFont fontWithName:@"Helvetica-Oblique" size:11.0],
                                    NSForegroundColorAttributeName : [NSColor grayColor] };
        NSDictionary *valAttrs = @{ NSFontAttributeName : [NSFont fontWithName:@"Helvetica-Bold" size:18.0] };
        NSAttributedString *newline = [[NSAttributedString alloc] initWithString:@"\n"];
        NSMutableAttributedString *output = [[NSMutableAttributedString alloc] init];
 
        // Iterate through pairs in the dictionary to add formatted output
        [localizationDict enumerateKeysAndObjectsUsingBlock:^(id key, id val, BOOL *stop) {
            NSAttributedString *keyString = [[NSAttributedString alloc] initWithString:key attributes:keyAttrs];
            NSAttributedString *valString = [[NSAttributedString alloc] initWithString:val attributes:valAttrs];
            [output appendAttributedString:valString];
            [output appendAttributedString:newline];
            [output appendAttributedString:keyString];
            [output appendAttributedString:newline];
            [output appendAttributedString:newline];
        }];
 
        // Get RTF representation of the attributed string
        NSData *rtfData = [output RTFFromRange:NSMakeRange(0, output.length) documentAttributes:nil];
 
        // Pass preview data to QuickLook
        QLPreviewRequestSetDataRepresentation(preview,
                                              (__bridge CFDataRef)rtfData,
                                              kUTTypeRTF,
                                              NULL);
    }
    return noErr;
}

```

## 生成丰富的HTML

动态生成预览的一种通常有用但稍微复杂的方法是创建HTML。这种方法非常适合本质上不是文本或图形的应用程序，例如其文档用户界面是文本和图形的组合的应用程序，或者以一个包含列表、文本、表单、标签复选框等控件用户界面来呈现文档数据的应用程序。

> 重要说明：出于安全原因，您不能使用传递回快速外观的HTML中的Web Kit插件（例如，不能使用Java applet，视频或Flash动画）。
> 

您也可以使用这种方法来转换以一种或多种非人类可读格式存储的文档，以实现更加用户友好的预览显示。清单6-2中的代码示例通过使用Foundation类读取属性列表文件并生成用于预览的格式化表示形式来说明这种用法。它还创建属性字典，以在调用`QLPreviewRequestSetDataRepresentation`时将其传递回Quick Look。该词典中的属性定义HTML数据以及与该数据关联的附件。

 <center>Listing 6-2  Generating a preview composed of HTML data plus an attachment</center>
 
 ```objc
 OSStatus GeneratePreviewForURL(void *thisInterface, QLPreviewRequestRef preview, CFURLRef url, CFStringRef contentTypeUTI, CFDictionaryRef options)
{
    @autoreleasepool {
 
        // Load the property list from the URL
        NSURL *nsurl = (__bridge NSURL *)url;
        NSData *data = [NSData dataWithContentsOfURL:nsurl];
        if (!data) return noErr;
        id plist = [NSPropertyListSerialization propertyListWithData:data options:NSPropertyListImmutable format:NULL error:nil];
        if (!plist) return noErr;
 
        // The above might have taken some time, so before proceeding make sure the user didn't cancel the request
        if (QLPreviewRequestIsCancelled(preview)) return noErr;
 
        // Set up the object that creates the HTML preview
        HTMLPreviewBuilder *builder = [[HTMLPreviewBuilder alloc] init];
        builder.title = [nsurl lastPathComponent];
        builder.stylesheet = [NSURL URLWithString:@"cid:plistStylesheet.css"];
 
        // Generate a preview for this plist and retrieve it as a string
        // (Begin recursive preview generation by wrapping the plist's top level object in a dictionary)
        NSString *html = [builder previewForDictionary:@{ @"Root" : plist }];
 
        // Load a CSS stylesheet to attach to the HTML
        NSBundle *bundle = [NSBundle bundleForClass:[HTMLPreviewBuilder class]];
        NSURL *cssFile = [bundle URLForResource:@"plistStylesheet" withExtension:@"css"];
        NSData *cssData = [NSData dataWithContentsOfURL:cssFile];
 
        // Put metadata and attachment in a dictionary
        NSDictionary *properties = @{ // properties for the HTML data
                                     (__bridge NSString *)kQLPreviewPropertyTextEncodingNameKey : @"UTF-8",
                                     (__bridge NSString *)kQLPreviewPropertyMIMETypeKey : @"text/html",
                                     // properties for attaching the CSS stylesheet
                                     (__bridge NSString *)kQLPreviewPropertyAttachmentsKey : @{
                                             @"plistStylesheet.css" : @{
                                                     (__bridge NSString *)kQLPreviewPropertyMIMETypeKey : @"text/css",
                                                     (__bridge NSString *)kQLPreviewPropertyAttachmentDataKey: cssData,
                                                     },
                                             },
                                     };
 
        // Pass preview data and metadata/attachment dictionary to QuickLook
        QLPreviewRequestSetDataRepresentation(preview,
                                              (__bridge CFDataRef)[html dataUsingEncoding:NSUTF8StringEncoding],
                                              kUTTypeHTML,
                                              (__bridge CFDictionaryRef)properties);
    }
    return noErr;
}
 ```
 > 注意：上面清单中的`HTMLPreviewBuilder`类是代码的占位符，该代码为包含有效的用于属性列表的NSArray或NSDictionary实例生成HTML表示形式。 它应该从顶级容器开始递归地遍历此类集合，并通过其stylesheet属性包括对URL集合的引用，以便可以使用CSS样式表来设置HTML表示的样式。
 
 清单6-2中有几件事值得特别注意：
 
 - `properties `字典包含HTML编码和HTML MIME类型（`kQLPreviewPropertyTextEncodingNameKey`和`kQLPreviewPropertyMIMETypeKey`）以及所有附件子词典。
 - HTML使用URL格式`cid:identifier`来引用CSS样式表附件。该标识符始终用作包含附件数据的字典的关键字，该附件数据包含在`properties `字典中。
- 有一个依附类词典； 它包含CSS样式表附件的MIME类型和样式表数据（可通过`kQLPreviewPropertyMIMETypeKey`和`kQLPreviewPropertyAttachmentDataKey`访问）。 
 - 当生成器调用`QLPreviewRequestSetDataRepresentation`时，它将传入HTML数据（以指定的编码）、属性字典和用于标识HTML内容的UTI常量。 通过以这种方式设置HTML和属性字典，WebKit可以加载HTML，并在对其进行解析时将附件加载到Web视图中。

 
# 7 Saving Previews and Thumbnails in the Document

作为向“快速查看”提供缩略图和预览数据的一种方法，应用程序可以将该数据存储为文档数据的一部分。然后，生成器可以访问它，并在对`QLThumbnailRequestSetImageWithData`或`QLPreviewRequestSetDataRepresentation`的调用中将其返回到快速查找。 这种方法可以为生成器提供快速响应时间，但要以较大的文档文件为代价。

为了说明您的生成器如何使用这种方法提供预览和缩略图，以下清单显示了对Sketch应用程序代码的修改，该应用程序将缩略图图像作为文档数据的一部分写入。清单7-1显示了如何定义`NSDocument`子类的属性来保存图像数据。

<center>Listing 7-1  Sketch example project: adding a thumbnail property</center>

```objc
@interface SKTDrawDocument : NSDocument {
    @private
    NSMutableArray *_graphics;
    // ...other instance variables here...
    NSData *_thumbnail;
}
// ...existing methods here...
- (NSData *)thumbnail;
```

实现`thumbnail `访问器方法以返回缩略图图像。 在准备将数据写出到文件中的`NSDocument`方法（`dataOfType: error:` ）中，添加了清单7-2中由`“new”`标签指示的代码行。

<center>Listing 7-2  Sketch example project: including the thumbnail with the document data</center>

```objc
static NSString *SKTThumbnailImageKey = @"SketchThumbnail";                   // new
 
- (NSData *)dataOfType:(NSString *)typeName error:(NSError **)outError {
    NSData *data,;
    NSArray *graphics = [self graphics];
    NSPrintInfo *printInfo = [self printInfo];
    NSWorkspace *workspace = [NSWorkspace sharedWorkspace];
    BOOL useTypeConformance = [workspace respondsToSelector:@selector(type:conformsToType:)];
 
    if ((useTypeConformance && [workspace type:SKTDrawDocumentNewTypeName conformsToType:typeName])
        || [typeName isEqualToString:SKTDrawDocumentOldTypeName]) {
        NSData *tiffRep;                                                     // new
        NSMutableDictionary *properties = [NSMutableDictionary dictionary];
 
        [properties setObject:[NSNumber numberWithInt:SKTDrawDocumentCurrentVersion] forKey:SKTDrawDocumentVersionKey];
        [properties setObject:[SKTGraphic propertiesWithGraphics:graphics] forKey:SKTDrawDocumentGraphicsKey];
        [properties setObject:[NSArchiver archivedDataWithRootObject:printInfo] forKey:SKTDrawDocumentPrintInfoKey];
        tiffRep = [self TIFFDataWithGraphics:graphics error:outError];        // new
        [properties setObject:tiffRep forKey:SKTThumbnailImageKey];           // new
        data = [NSPropertyListSerialization dataFromPropertyList:properties
                                                          format:NSPropertyListBinaryFormat_v1_0
                                                errorDescription:NULL];
    } else if ((useTypeConformance && [workspace type:(__bridge NSString *)kUTTypePDF conformsToType:typeName])
               || [typeName isEqualToString:NSPDFPboardType]) {
        data = [SKTRenderingView pdfDataWithGraphics:graphics];
    } else {
        NSParameterAssert((useTypeConformance && [workspace type:(__bridge NSString *)kUTTypeTIFF conformsToType:typeName])
                          || [typeName isEqualToString:NSTIFFPboardType]);
        data = [SKTRenderingView tiffDataWithGraphics:graphics error:outError];
    }
    return data;
}
```

在用于读取文档数据的相应`NSDocument`方法(`readFromData: ofType: error:`)中，将缩略图从文档属性字典中中“解包”：

```objc
_thumbnail = [properties objectForKey:SKTThumbnailImageKey];

```

现在，实现Sketch的生成器很简单，只需访问缩略图图像数据并将其传递给`QLThumbnailRequestSetImageWithData`调用中的Quick Look，如清单7-3所示。 （对于预览，相应的功能是`QLPreviewRequestSetDataRepresentation`。）

<center>Listing 7-3  Returning the stored thumbnail image to Quick Look</center>

```objc
OSStatus GenerateThumbnailForURL(void *thisInterface, QLThumbnailRequestRef thumbnail, CFURLRef url, CFStringRef contentTypeUTI, CFDictionaryRef options, CGSize maxSize)
{
    @autoreleasepool {
        SKTDrawDocument* document = [[SKTDrawDocument alloc] init];
        if (![document readFromURL:(__bridge NSURL *)url
                            ofType:(__bridge NSString *)contentTypeUTI]) {
            return noErr;
        }
        if ([document respondsToSelector:@selector(thumbnail)]) {  // runtime verification
            NSData *tiffData = [document thumbnail];
            if (tiffData != nil) {
                NSDictionary *props = [NSDictionary dictionaryWithObject:@"public.tiff" forKey:(__bridge NSString *)kCGImageSourceTypeIdentifierHint];
                QLThumbnailRequestSetImageWithData(thumbnail, (__bridge CFDataRef)tiffData, (__bridge CFDictionaryRef)props);
                return noErr;
            }
        }
        NSSize canvasSize = [document canvasSize];
        CGContextRef cgContext = QLThumbnailRequestCreateContext(thumbnail, *(CGSize *)&canvasSize, false, NULL);
        if (cgContext) {
            NSGraphicsContext* context = [NSGraphicsContext graphicsContextWithGraphicsPort:(void *)cgContext flipped:YES];
            if (context) {
                [document drawDocumentInContext:context];
            }
            QLThumbnailRequestFlushContext(thumbnail, cgContext);
            CFRelease(cgContext);
        }
    }
    return noErr;
}
```

在对`QLThumbnailRequestSetImageWithData`的调用中，生成器使用`kCGImageSourceTypeIdentifierHint` 属性将图像格式指示为“快速查看”。 请注意，此示例检查文档对象的类是否实现了缩略图访问器方法（以排除应用程序的先前版本），如果是，则检查是否返回了缩略图数据。 如果不是，它将在“快速查看”提供的图形上下文中绘制缩略图。

# 8 Assigning Core Graphics Images to Thumbnails 

在某些情况下，生成缩略图的最简单方法是创建Core Graphics图像，而不是创建与I/O框架兼容的图像或在图形上下文中绘制该图像。例如，您的生成器可能使用的框架可以直接提供缩略图的序列化版本作为CGImage对象。 对于这种情况，您可以创建Core Graphics图像，然后通过调用`QLThumbnailRequestSetImage`函数将该图像传达给Quick Look。 此功能与`QLThumbnailRequestSetImageWithData`之间的主要区别在于，后者功能要求图像数据采用I/O框架支持的格式。


清单8-1说明了使用QT Kit框架的方法将电影的海报帧作为Core Graphics图像并将其设置为电影文件的缩略图的方法。

<center>Listing 8-1  Creating and assigning a Core Graphics image</center>

```objc
OSStatus GenerateThumbnailForURL(void *thisInterface, QLThumbnailRequestRef thumbnail, CFURLRef url, CFStringRef contentTypeUTI, CFDictionaryRef options, CGSize maxSize)
{
    NSError *theErr;
    QTMovie *theMovie = [QTMovie movieWithURL:(__bridge NSURL *)url error:&theErr];
    if (theMovie == nil) {
        if (theErr != nil) {
            NSLog(@"Couldn't load movie URL, error = %@", theErr);
        }
        return noErr;
    }
    [theMovie gotoPosterTime];
    QTTime mTime = [theMovie currentTime];
    NSDictionary *imgProp = [NSDictionary dictionaryWithObject:QTMovieFrameImageTypeCGImageRef forKey:QTMovieFrameImageType];
    CGImageRef theImage = (__bridge CGImageRef)[theMovie frameImageAtTime:mTime withAttributes:imgProp error:&theErr];
 
    if (theImage == nil) {
        if (theErr != nil) {
            NSLog(@"Couldn't create CGImageRef, error = %@", theErr);
        }
        return noErr;
    }
    QLThumbnailRequestSetImage(thumbnail, theImage, NULL);
    return noErr;
}
```

# 9 Canceling Previews and Thumbnails 

户端应用程序或Quick Look守护程序（`quicklookd`）可能会决定不再需要生成器请求的预览图像或缩略图。通常，发生这种情况是因为用户已指示（例如，通过关闭Finder窗口）他们对所列文档不感兴趣。当客户端或`quicklookd `决定不再需要缩略图或预览时，“快速浏览”会以两种方式通知相应的生成器，以下各节将对此进行介绍。生成器应查找此取消，停止任何正在进行的图像生成，并清除用于生成预览或缩略图的所有资源。

## 通过回调函数取消

当客户端应用程序不再需要其已请求的缩略图或预览时，它会告诉Quick Look，后者将根据先前请求的项的类型调用两个回调函数之一：
- `CancelThumbnailGeneration` 用于取消缩略图的生成
- `CancelPreviewGeneration` 用于取消预览的生成

生成器可以实现这些功能，以停止创建预览和缩略图，并清除到目前为止在其创建中使用的所有资源，并尽快返回。

但是，通常不建议你的代码通过实现这些回调函数之一来取消预览和缩略图的生成。由于Quick Look总是在**辅助线程**中调用这些函数，因此安全地实现它可能很困难。 例如，您必须小心地将取消请求与图像生成中涉及的线程进行匹配。 如果对确保代码的线程安全性有任何疑问，请遵循《通过轮询取消》中描述的指导方针。


## 通过轮询取消

当客户端应用程序不再需要其已请求的缩略图或预览时，它会告诉Quick Look，后者将为该请求设置一个bool标志（除了调用`CancelThumbnailGeneration`或`CancelPreviewGeneration`回调函数）。生成器可以通过调用`QLThumbnailRequestIsCancelled`函数（用于缩略图）或`QLPreviewRequestIsCancelled`函数（用于预览）随时访问此标志的值。

在生成器代码中，您可以定期调用这些函数以轮询“快速查找”以了解当前请求的取消状态。如果调用返回的是true，请清除到目前为止在生成缩略图或预览时使用的所有资源，然后返回`noErr`。对于大多数生成器，建议使用此方法，而不要使用《通过回调函数取消》中介绍的方法。


您应该在生成器代码中的适当位置调用`QLThumbnailRequestIsCancelled`和`QLPreviewRequestIsCancelled`。哪个位置合适取决于代码在做什么以及代码的构造方式。通常，您应该在执行一些耗时的任务之前测试请求取消，尤其是当您无法在任务执行过程中查询取消状态时（例如，解析文件）

这有个很有用的例子。典型生成器的逻辑在其GeneratePreviewForURL回调函数中具有以下结构：


- 加载文档数据。
-  解析文档数据。
-  合成预览或将其转换为本地“快速查看”类型。
-  刷新图形上下文或在响应中设置数据。

在这种结构下，您可能应该在第1步和第2步之间以及在第2步和第3步之间再次调用`QLPreviewRequestIsCancelled`。您无需在第3步和第4步之间调用该函数，因为在完成第4步之后，“快速查找”将丢弃预览。 重要的是明智地轮询取消； 您不应该经常轮询，但是同时您应该经常轮询，以确保取消的预览或缩略图不会影响性能。

# 10 Debugging and Testing a Generator


 快速查看为开发人员提供了一些调试和测试其生成器代码的工具。以下各节描述了这些功能，并提供了一些调试和测试生成器的策略和建议。
 
 ## 调试工具
 
 因为生成器是一个插件，而不是一个自包含的可执行文件，所以如果您独自一人，调试它可能会出现问题。 幸运的是，Quick Look提供了一种轻松调试生成器代码的方法：`qlmanage`诊断工具（安装在`/usr/bin`中）。 `qlmanage`在几乎与Quick Look守护程序（`quicklookd `）相同的环境中执行项目的生成器。 您可以将该工具作为项目的可执行文件运行，并且通过指定某些参数，可以逐步查看生成器代码，并查看其如何处理预览和缩略图。
 
 要设置用于调试的Quick Look项目，请完成以下步骤：
 
# 补充资料
 
 [Mac OS X平台下QuickLook开发教程](https://www.cnblogs.com/csuftzzk/p/how_to_make_quick_look_plugins.html)
 
 [macOS QuickLook常用插件](https://www.jianshu.com/p/7421f7a03e07)
 
 [Uniform Type Identifiers](https://escapetech.eu/manuals/qdrop/uti.html)
 
 [常见 MIME 类型列表](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Basics_of_HTTP/MIME_types/Complete_list_of_MIME_types)
 