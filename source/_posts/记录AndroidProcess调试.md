---
title: 记录AndroidProcess调试
date: 2017-05-26 15:48:42
tags: Android
---

记录AndroidProcess调试步骤

**第一步**在AnnoProcessor中设置断点

![](http://i.imgur.com/gWBxFFT.png)
 
<!-- more -->

> 记录AndroidProcess调试步骤
> 
> **第一步**在AnnoProcessor中设置断点
> 
> ![](http://i.imgur.com/gWBxFFT.png)
>
> **第二步**修改
> 
> gradle.properties
> 
> **第三步**添加如下代码
> 
> 第一行的作用是允许gradle开启守护进程
> 
> 第二行代码是设置gradle连接的端口号.这里原先是设置为5005，但是由于报错，说端口号被占用，因此改为5010
> 
> ``org.gradle.daemon=true``
> 
> ``org.gradle.jvmargs=-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=5010``
>
> **第四步**在上一步的基础上在as控制台运行如下命令：
> 
> ``gradle --daemon``
> 
> 你可能会有如下的报错：
> 
> ![](http://i.imgur.com/gMRzcuZ.png)
>
> **少年莫慌，这是因为gradle没有在你的电脑中配置的缘故，因此可以将gradle改成gradlew！！！！！**
> 
> gradlew相当于是gradle的一个包装，相当于接口，因此实现类是由，.gradle文件夹中的gradle-wrapper.properties中的distributionUrl后面指定的gradle版本决定的**（PS:在这里可以看到面向接口的好处了吧）**
> 
> 运行完成之后，gradle就已经开启了守护进程，他会监听在我们刚刚配置的指定的端口号上
> 
> **第五步**配置AndroidStudio
> 
> ![](http://i.imgur.com/lmCs3Wd.png)
> 
> ![](http://i.imgur.com/B1tGwYV.png)
> 
> ![](http://i.imgur.com/y0vgumU.png)
> 
> **然后点击ok就可以了**然后界面上就有这个配置项了，下面开始运行它，点击那个小虫子的图标，debug模式运行
> 
> ![](http://i.imgur.com/RQNGkba.png)
> 
> ![](http://i.imgur.com/hmokmXh.png)
> 
> **第五步**最后一步啦，在命令行执行如下命令，让其进行编译，因此我们
> 
> ``gradle clean assembleDebug``
>
> 在运行的过程中
> 
> ![](http://i.imgur.com/jivxUeh.png)
> 
> 这里可以看到我们的断点执行啦！！ok到此AnnotationProcessor的调试就到此结束了~！