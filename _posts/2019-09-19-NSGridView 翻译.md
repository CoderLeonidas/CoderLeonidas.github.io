---
layout:     post
title:      "NSGridView翻译"
// subtitle:   " \"Hello World, Hello Blog\""
date:       2019-09-19 22:38:02
author:     "CoderLeonidas"
catalog: true
tags:
- 翻译
- Cocoa

---

# NSGridView 翻译

> 网格视图，一个能在一个灵活的行列网格中对齐视图的容器。

--- 

### 定义

```
@interface NSGridView : NSView

```

### 概述

网格视图可帮助您以类似于电子表格的行列排列方式布置内容，例如照片或缩略图。 在网格视图中，占用单个行 - 列交集的项由NSGridCell对象表示。

### 话题

## 创建一个GridView

`+ gridViewWithNumberOfColumns:rows:`
> 创建具有指定列数和行数的新分配的网格视图对象。

`+ gridViewWithViews:`
> 使用指定的视图数组数组创建新分配的网格视图对象。
> 此方法创建一个自动释放的网格视图，其大小足以容纳传递的行数组。 数组中的每个元素本身都是该行的视图数组。

`- initWithFrame:`
> 使用指定的框架矩形创建新分配的网格视图对象。

`- initWithCoder:`
> 从编码器创建新分配的网格视图对象。


## 获取网格信息

`numberOfRows`
> 行数

`numberOfColumns`
> 列数

`- indexOfColumn:`
> 返回指定列的索引

`- rowAtIndex:`
> 返回指定索引所在的行对象

`- columnAtIndex:`
> 返回指定索引所在的列对象


`- indexOfRow:`
> 返回指定行的索引


## 添加、删除、移动行

`- addRowWithViews:`
> 添加一组视图到一个新的行。
> 你可以在girdview里动态的插入或者移除行和列。网格会根据需要放大以保留指定的视图。

`- insertRowAtIndex:withViews:`
> 将视图对象数组插入索引处的网格视图中。

`- removeRowAtIndex:`
> 从索引处的网格视图中删除行。

`- moveRowAtIndex:toIndex:`
> 将指定的行移动到新的行位置。


## 添加、删除、移动列
`- addColumnWithViews:`
> 添加包含视图数组的新列。

`- insertColumnAtIndex:withViews:`
> 在指定的索引处插入视图对象数组。

`- removeColumnAtIndex:`
> 从指定索引处的网格视图中删除列。

`- moveColumnAtIndex:toIndex:`
> 将指定列移动到新列位置。



## 管理网格间距和对齐
`NSGridViewSizeForContent `
> 行和列大小的默认值。
> 此常量表示行或列应自动适合内容视图。

`columnSpacing`
> 网格视图的列间距。

`rowSpacing`
> 网格视图的行间距。

`rowAlignment`
> 网格视图的行对齐方式。

`xPlacement`
> 单元格在网格列中的位置。

`yPlacement`
> 单元格在网格行中的位置。


## 创建、合并单元格

`- cellAtColumnIndex:rowIndex:`
> 返回指定列和行索引处的网格单元对象。

`- cellForView:`
> 返回包含给定视图或其祖先之一的网格单元对象。

`- mergeCellsInHorizontalRange:verticalRange:`
> 在单元格左上角进行水平和垂直范围内进行扩展以包含整个区域
> 
> 此函数使范围中的其他单元格无效，并且它们不再维护其布局，约束或内容视图。 单元格合并对网格视图的基本单元坐标系没有影响，合并区域内的单元格引用指的是单个合并单元格。

> 在安装视图之前，使用此方法配置网格几何体。 如果要合并的单元格包含内容视图，则仅保留最顶层的视图。
