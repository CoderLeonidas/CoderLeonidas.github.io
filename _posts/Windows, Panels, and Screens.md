# Windows, Panels, and Screens

组织您的视图层次结构，并促使其在屏幕上的显示。

## 主题

### 窗口

#### NSWindow

应用程序在屏幕上显示的窗口。

#### NSPanel

一种特殊类型的窗口，通常执行主窗口的辅助功能。

#### NSWindowDelegate

一组可选的方法，NSWindow的委托可以实现这些方法来响应事件，例如窗口调整大小、移动、暴露和最小化。

#### NSWindowTab

与属于选项卡组的窗口相关联的选项卡。

### NSWindowTabGroup

作为单个选项卡窗口一起显示的一组窗口。

### 窗口复原

#### NSWindowRestoration

复原类必须实现的一组方法来处理窗口的重新创建。

#### NSUserInterfaceItemIdentification

一组用于将唯一标识符与用户界面中的对象关联的方法。

### 屏幕

#### NSScreen

描述计算机显示器或屏幕属性的对象。

### 弹出窗口

#### NSPopover

一种在屏幕上显示与现有内容相关的附加内容的方法。

#### NSPopoverDelegate

弹出窗口委托可以实现的一组可选方法，以提供附加或自定义功能。


### 警告

#### NSAlert

附加到文档窗口的模态对话框或工作表单。

#### NSAlertDelegate

由NSAlert对象的委托实现的一组可选方法，用于响应用户的求助请求

### 打开和保存面板

#### NSOpenPanel

提示用户选择要打开的文件的面板。

#### NSSavePanel

提示用户有关文件保存位置的信息的面板。

#### NSOpenSavePanelDelegate

一组用于管理与打开或保存面板的交互的方法。

### 打印和PDF面板

#### NSPDFPanel

保存或导出为PDF的面板，与macOS用户界面一致。

#### NSPrintPanelAccessorizing

打印面板对象可用于从打印附件控制器获取信息的一组方法。

### 颜色面板

#### NSColorPanel

用于在应用程序中选择颜色的标准用户界面。

#### NSColorPickingCustom

一组方法，提供了一种方法来添加颜色选择器自定义用户界面的颜色选择到一个应用程序的颜色面板。

#### NSColorPickingDefault

为颜色选择器提供基本行为的一组方法。

#### NSColorPicker

实现NSColorPickingDefault协议的抽象超类。


### 字体面板

#### NSFontPanel

字体面板是显示可用字体列表的用户界面对象，允许用户预览这些字体并更改用于显示文本的字体。

#### NSFontPanelModeMask

#### NSFontPanelValidation

#### NSFontChanging

用于告诉字体面板显示其部分或全部元素的一组方法。




## 参考

[Windows, Panels, and Screens](https://developer.apple.com/documentation/appkit/windows_panels_and_screens?language=objc)