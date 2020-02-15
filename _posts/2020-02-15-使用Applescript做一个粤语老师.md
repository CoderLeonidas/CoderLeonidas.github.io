---
layout:     post
title:      "使用Applescript做一个粤语老师"
date:       2020-02-15 16:08:02
author:     "CoderLeonidas"
catalog: true
tags:

- Cocoa
- macOS
- 系统工具


---


# 使用Applescript做一个粤语老师



这个其实没啥技术含量，纯粹觉得好玩而已。直接上步骤：

## 使用步骤

-  1 打开**ScriptEditor**

-  2 键入以下代码:

```applescript
tell current application	say "抬望眼，仰天长啸，壮怀激烈" using "Sin-Ji"	say "人没有梦想，跟夏侯惇有什么区别" using "Sin-Ji"	say "我心中的一团火是不会熄灭滴" using "Sin-Ji"	end tell
```

-  3 点击run按钮
-  ![](https://tva1.sinaimg.cn/large/0082zybply1gbx5fdgi1mj30qw0gstdw.jpg)

- 4 Enjoy

## 简单代码说明

- `tell current application`: 告诉ScriptEditor，你该干活了
- `say`: 让ScriptEditor朗读
- `xxx`: 朗读的内容，可以替换为你自己的文本。想知道哪句话粤语怎么讲，直接往里替换文本内容或者添加一行
- `using "Sin-Ji"`: 朗读时使用谁的语音声音。"Sin-Ji"，是一个粤语的女生语音包。当然，还可以使用其他的[语音包](http://www.443w.com/tts/?tag=语音包)


## 结语
很简单吧，来试试，把《海阔天空》的歌词导入进来，下次KTV上给朋友们秀一把吧哈哈~




