---
title: React-Native原理的简单讨论
date: 2018-11-04 09:46:34
tags: react native
desc: React-Native原理的简单讨论
---

> 文章中的代码基于 ract-native 0.57.1进行编写

<img src="https://i.imgur.com/wdMolXp.png" width="680px" />

<h4>Android</h4>

- interface ReactApplication 
该类用于获取一个默认的ReactHost实例对象

- class ReactHost
包含ReactInstanceManager对象的一个简单的类

- **class ReactInstanceManager**
该类用于管理CatalystInstance实力对象，配合ReactRootView管理View的创建与生命周期等功能。

- **class ReactRootView**
ReactRootView是Catalyst的主视图，提供侦听大小更改的功能，以便UI管理器可以重新布局其元素。它委托处理自身和子视图的触摸事件，并使用JSTouchDispatcher将这些事件发送给JS。

- **class CatalystInstance**
用于js与原生之间的通信管理类

- **JavaScriptModule/NativeModule**
JavaScriptModule是JS Module，负责JS到Java的映射调用格式声明，由CatalystInstance统一管理。<br/>NativeModule是ava Module，负责Java到Js的映射调用格式声明，由CatalystInstance统一管理。<br/>AppRegistry、DeviceEventEmitter通过继承JavaScriptModule，然后声明相关方法即可调用。例如通过reactContext.getJSModule AppRegistry.class 方法获取AppRegistry对象即可调用js端暴露的方法。通过继承NativeModule然后编写方法并添加**@ReactMethod**注解即从native端暴露给js端调用。
<!-- more -->

<h4>React-Native是如何将一个jsx元素映射为一个原生控件的？</h4>
1.加载<img src="https://i.imgur.com/j3ei8P8.png" width="680px" />
2.渲染<img src="https://i.imgur.com/hlhPWSA.png" width="680px" />
<h4>渲染原理</h4>
> 编写jsx通过babel转换成标准js代码，通过createElement创建element

> UIManager.js通过cpp端传递事件给Native端（Android/Ios）,利用了UIManagerModule.java中的createView、updateView、manageChildren等方法进行界面中元素的修改

> UIManagerModule负责从js传递来的修改element元素的这些操作封装成事件，并添加到队列中，最终完成渲染
