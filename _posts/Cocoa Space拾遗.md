# Cocoa Space拾遗


## 一个需求

**跨进程窗口同步**:  B进程的窗口需要始终盖在A进程的窗口上面。A窗口出现的地方，就应该有B窗口。

-  一种做法：B进程获得A进程窗口所在的SpaceID，然后将B进程窗口也move到A进程所在的Space上。

## Space的零碎概念

- Mac的屏幕工作空间
- 分为2种: 全屏空间和用户空间(非全屏)
- 用户空间每个屏幕最多可以放16个(Note: 双屏时拔出显示屏，会多于16个，数量是原先的总个数-1)
- 双屏下，每个屏幕的工作空间可以选择独立或者同步
- 每个空间都有一个ID
- 窗口\空间\屏幕之间存在的关系:
 - 屏幕可以包含多个空间，一对多
 - 空间可以包含多个窗口，一对多
 - 可以指定窗口在哪个空间上(窗口可以在多个空间上)，一对多
 - 窗口和屏幕的关系是确定的，一对多


## 相关设置

![](https://tva1.sinaimg.cn/large/0081Kckwly1gkt86zn39oj31140qotdn.jpg)

## 相关plist
```
~/Library/Preferences/com.apple.spaces.plist
```
![](https://tva1.sinaimg.cn/large/0081Kckwly1gkrafcedzuj30go075t9e.jpg)


## 相关API

-  开放API：已经从 `CGWindow.h` 废除
 
 ![](https://tva1.sinaimg.cn/large/0081Kckwly1gkrahhsn7wj30ju0cudil.jpg)

- 私有API：仍然能用，可从三方库[phoenix](https://github.com/kasper/phoenix)里窥探到。

 
## 应用

- Meeting share 的内容颗粒度可以更细，只分享某个Space的内容而非某个屏幕上的所有Space的内容
- 跨进程同步窗口

## Demo
 

## 参考资料

[在 Mac 上的多个空间中工作](https://support.apple.com/zh-cn/guide/mac-help/mh14112/mac)

[phoenix](https://github.com/kasper/phoenix)

