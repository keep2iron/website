---
title: okhttp源码解析
date: 2018-04-29 10:41:03
tags:
---

![](https://i.imgur.com/pmIxXFe.png)

安卓开发领域上[Okhttp](https://github.com/square/okhttp)、[Retrofit](https://github.com/square/retrofit)、[RxJava](https://github.com/ReactiveX/RxJava)无疑已经成为了开发过程中的三板斧。在安卓6.0中谷歌删除了HttpClient的相关代码，并且在系统层面上使用了OkHttp，因此足以说明了这个框架的优秀。

[我们不重复造轮子不表示我们不需要知道轮子该怎么造及如何更好的造!](https://github.com/android-cn/android-open-project-analysis) 

<!-- more -->

用了OkHttp已经一年多之后，现在是时候来一探究竟了!!

## 整体思路
