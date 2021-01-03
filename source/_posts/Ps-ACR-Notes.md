---
title: Adobe Camera Raw Notes
tags:
  - Photoshop
  - Note
categories: 「快门」
permalink: Ps-ACR-Notes
date: 2020-03-10 23:42:06
---

<center> <font color="#bababa">

***思路跟 Lr 一样的***

</font></center>
<!--more-->

---

# 调整

## 获得通透的画面

- `通透`：增加细节
- 途径：亮度再分配
- 关键：把画面中的大部分细节调整到人眼可见的亮度范围内
- tips：调亮主影调、阴影的时候还可以尝试增加黑场（压低黑色）
- [范例视频](https://www.youtube.com/watch?v=sYj8cMlecJE&list=PLhnwj_CftHvhCdkgMKU8qAL4GlXaXmZ1o&index=3)

## 亮度再分配

- 工具：`ACR`中的基本调整滑块
- 原则：
  - 确保画面有正确的黑白场
  - 提升暗部细节（在确保有黑场的条件下）
  - 压暗亮部细节（在确保有白场的条件下）
- 目的：获取可感知的细节，让细节落入人眼的可视范围内
-  [讲解视频](https://www.youtube.com/watch?v=bcy9MFub09o)

## 精确白平衡

- 色温和色调可以确定一个点的颜色，相当于一对坐标
- `色温`：蓝色 ↔ 黄色
- `色调`：绿色 ↔ 洋红色
- 白平衡工具：
    + 吸取画面中原本应该是白色（中性灰：纯白-纯黑之间都可以）的点
    + 原理：补色的原理。如果画面中的白色准了，那认为其他颜色也准了。
    + tips：尽量不要选白纸（有荧光粉）、眼睛（亚洲人、动物偏黄）
- JPG格式的白平衡调整色彩不可逆
- [范例视频](https://www.youtube.com/watch?v=5oEbQ6m2O5w&list=PLhnwj_CftHvhCdkgMKU8qAL4GlXaXmZ1o&index=4)

## 精确调色

- `饱和度`：
    + `自然饱和度`：用得多，更智能，会识别图中内容
    + `饱和度`：全部调，不建议
- `色调分离`：
    + 大部分电影：高光偏黄，阴影偏蓝 
- `HSL`：调节不同颜色的色相、饱和度、明亮度
    + `灰度混合`：调整不同颜色会黑白照片亮度贡献的大小
- [范例视频](https://www.youtube.com/watch?v=2fde25zgXb8&list=PLhnwj_CftHvhCdkgMKU8qAL4GlXaXmZ1o&index=5)

## 渐变滤镜

- 硬件：是一种灰度过度滤镜，用于控制天空
- 可适用于大光比的照片
- 可以渐变得使亮度再分配
- eg：
    + 天空压暗、降低色温
    + 地面提亮、暖色调、通透
- 渐变滤镜的蒙版`（范围遮罩）`：
    + 颜色
    + 明亮度
    + 按住`Alt`可以看到蒙版作用范围：越亮→作用越强；越暗→作用越弱
- [范例视频1](https://www.youtube.com/watch?v=l365G5c65kE&list=PLhnwj_CftHvhCdkgMKU8qAL4GlXaXmZ1o&index=6)、[范例视频2](https://www.youtube.com/watch?v=OdlCFo5bNBU&list=PLhnwj_CftHvhCdkgMKU8qAL4GlXaXmZ1o&index=9)

## 综合应用

- 去除紫边
    + `镜头校正`-`去边`-去除紫边或者绿边
- 渐变滤镜
- 径向滤镜
    + 内部/外部
- 亮度再分配
- 色彩快速调节
- 透视畸变矫正
- ACR二次调用
- [范例视频](https://www.youtube.com/watch?v=_b6KALfwJyE&list=PLhnwj_CftHvhCdkgMKU8qAL4GlXaXmZ1o&index=7)

## 用亮度蒙版和ACR恢复天空细节

- 目的：压暗天空，减少对近景的副作用，过度更自然
- `亮度蒙版`：借助画面原有的亮度信息来产生蒙版，通常由通道产生
- 选择主体、背景差别大的通道
- `亮度蒙版`操作：
    + 按住`Ctrl`并点击通道，创建亮度选区
    + 蚁行线：亮度高于50%的选中
    + 点回RGB通道，回到图层原始画面
    + 保持选区不变，`调整`-`曲线`，此时曲线的蒙版就是刚选中的亮度蒙版
    + 若有的近景受到了影响（如饱和度降低），可以双击蒙版，调整蒙版的羽化值，模糊蒙版
- ACR`去除薄雾`滑块：
    + 最有效的恢复天空细节的工具
    + 还可以搭配滤镜加强调整
    + 其他地方可以用亮度蒙版遮挡，不用担心对前景的影响
    + 副作用：可能改变天空的颜色，需要调整`（自然）饱和度`
- [范例视频](https://www.youtube.com/watch?v=7N7A0H7y4es&list=PLhnwj_CftHvhCdkgMKU8qAL4GlXaXmZ1o&index=10)

---

# 基础知识

## RAW 格式

- RAW：未经处理的原始数据
- 丰富的层次：16-bit
- 宽容度牛逼
- 私有色彩空间
- 没有设定黑白场
- 若操作`.jpg`，可以`图像`-`模式`-`16位通道`
- [范例视频](https://www.youtube.com/watch?v=yZEp7wWqswA&list=PLhnwj_CftHvhCdkgMKU8qAL4GlXaXmZ1o&index=1)

## ACR 简介

- 调用方式
    + 直接打开RAW文件
    + `滤镜`-`Camera Raw滤镜`
- 重要设置
    + 最下放的链接：`色彩深度`设定成16位
    + 色彩空间：需要由是否全链路色彩管理决定
- 无损调节：配置存放于`.xmp`文件中
- [范例视频](https://www.youtube.com/watch?v=H5SBJcz2bVM&list=PLhnwj_CftHvhCdkgMKU8qAL4GlXaXmZ1o&index=2)