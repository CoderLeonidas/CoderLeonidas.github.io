# VDI项目备忘录

1 windows VDI项目中log文件的目录：


```c++
C:\Users\admin\AppData\Roaming\ZoomVDI\logs\citrix_20200320_174535_19692.log

C:\Users\admin\AppData\Local\VirtualStore\Program Files (x86)\Citrix\ICA Client\debug.log
```

2 日至相关的宏定义：

```c++
#define ZLOG(severity,msg)	LAZY_STREAM(LOG_STREAM(severity), LOG_IS_ON(severity)) << msg << " "

#define LAZY_STREAM(stream, condition)   !(condition) ? (void) 0 : ::logging::LogMessageVoidify() & (stream)

#define LOG_STREAM(severity) COMPACT_GOOGLE_LOG_ ## severity.stream()

#define LOG_IS_ON(severity) ((::logging::LOG_ ## severity) >= ::logging::GetMinLogLevel())
```

3 日志路径函数

CmmLoggingFile.cc: `GetDefaultLogFile`

driver.cpp:  `initLog(TEXT("citrix"));`

4 日志文件写入方式 

debug.log： 由`ZLOG`和`DebugOuput` 写入

citrix_20200320_174535_19692.log： 由`ZLOG` 写入


5 Mac日志文件目录

6  重要的宏：

```c
BUILD_FOR_VDI
__VDI__
```
在Build settings中 Preprocessor Marcos 里配置

7 插件动态库cmmlib链接失败导致的加载失败： 

```

```

解决方法是，在cmmlib的build setting里将rpath删除。




8 术语:

sink

channel

ChannelSink

ChannelEventSink

stream

ChannelBundle


9  channel接口

CitrixICAChannel.h是对citrix的接口

IMediaConfChannel.h 是对media进程的接口

10 需要适配的宏定义：

```c

#ifdef __LINUX_CLIENT__
#if defined (__LINUX_CLIENT__)

```

#define __MACOS__
#define macintosh
#define __MAC_CLIENT__
#define CMM_MAC
#define WM_APP 





## 疑问

1 FindMediaBinPath 是什么路径？

2 ICADataArrival 写钩子的实现


