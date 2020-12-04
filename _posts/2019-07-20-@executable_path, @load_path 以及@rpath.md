# @executable\_path, @load\_path 以及@rpath



## 绝对路径

对于安装在共享位置的framework非常有用。比如：

安装路径: ` /Library/Frameworks/Foo.framework/Versions/A/Foo`

## @executable_path

对于嵌入在应用程序内部的Framework很有用，因为它允许您指定Framework对于应用程序可执行文件的位置：

* 安装路径： ` @executable_path/../Frameworks/Foo.framework/Versions/A/Foo`
* 应用程序位置：` /Applications/Foo.app` 
* 可执行文件路径：`/Applications/Foo.app/Contents/MacOS`
* Framework位置：`/Applications/Foo.app/Contents/Frameworks/Foo.framework`

链接器将所有这些放在一起，以找出可以在`/Applications/Foo.app/Contents/MacOS/../Frameworks/Foo.framework/Versions/A/Foo`处找到的Framework二进制文件.


## @loadear_path

从Mac OS X 10.4 Tiger起可用； 这对于嵌入插件内部的Framework很有用，因为它允许您指定Framework相对于**plug-ins**代码的位置（请记住，**plug-ins**,可能实际上并不知道它们相对于应用程序的安装位置， 因此，在这种情况下，知道`@executable_path`不会对我们有帮助）：


* 安装路径： `  @loader_path/../Frameworks/Foo.framework/Versions/A/Foo`
* 应用程序位置：` /Applications/Foo.app` 
* Plug-in位置：`/Library/Application Support/Foo/Plug-Ins/Bar.bundle`
* 可执行文件路径：`/Applications/Foo.app/Contents/MacOS`
* Loader path： `/Library/Application Support/Foo/Plug-Ins/Bar.bundle/Contents/MacOS`
* Framework位置：`/Library/Application Support/Foo/Plug-Ins/Bar.bundle/Contents/Frameworks/Foo.framework`

链接器将所有这些放在一起，以找出可以在`/Library/Application Support/Foo/Plug-Ins/Bar.bundle/Contents/MacOS/../Frameworks/Foo.framework/Versions/A/Foo`处找到的Framework二进制文件


请注意，如果“loader”是应用程序而不是**Plug-In**，则`@loader_path`最终等同于`@executable_path`。


## @rpath

Mac OS X 10.5 Leopard中的新功能是**@rpath**。 关键点：

@rpath指示动态链接器搜索路径列表以查找Framework
至关重要的是，此列表已嵌入到正在加载的应用程序中

这意味着有着`@rpath/Foo.framework/Versions/A/Foo `的单个Framework可以以多种不同的方式工作。也就是说，不用再使用`@executable_path`或`@loader_path`来指定安装路径。

## 参考

[@executable path, @load path and @rpath](http://www.cppblog.com/lauer3912/archive/2012/04/18/171814.html)



