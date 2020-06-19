---
title: Android绘制流程
date: 2019-03-31 22:39:47
tags: UI
---

# What

绘制流程的三个核心函数**onMeasure（测绘控件的大小）**、**onLayout（设置控件相对于父布局的位置）**、**onDraw（控件要绘制什么样的东西）**，以上的三个步骤全部都是在UI线程中完成操作，今天就主要来进行讨论这三个函数在Android系统的中调用逻辑。

>  由于60fps帧率的需要，又由于需要在1秒内(1s = 1000ms)完成刷新60帧的操作，因此得出以下结论
>  
>  1000 / 60 = 16.667 ms/fps
> 
>  Android会每隔16.667ms进行绘制，因此我们的绘制流程需要在这段时间内完成，不然就会产生我们俗称的卡顿或者ANR（Application Not Responding）。

# Why

那么为什么要进行设计这三个流程呢？这就要从Android设计中独有的宽高单位中的**Match_Parent、Wrap_Content、声明式大小、**三种设计有关系。Match_Parent和Wrap_Content是设置控件大小的两种参数后面简称为Match和Wrap。

#### onMeasure

Match是填充父控件的大小、Wrap是进行自适应包裹，这两者我们在使用时，例如像LinearLayout等一些控件时发现它的大小系统自动帮我们计算好了，而宽高大小的计算流程便是写在了**onMeasure**中。

#### onLayout

例如：LinearLayout通过设置Orientation属性之后进行item在一定方向上的有序排列。FrameLayout则是通过margin、layout_gravity进行设置item针对父布局的层级排列。这些item在布局中逻辑就是在**onLayout**中完成的。

#### onDraw

View中的空实现由各个集成自View的控件各自实现逻辑完成。

# How
