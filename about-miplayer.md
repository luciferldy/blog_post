+++
date = "2016-08-24T23:40:59+08:00"
menu = "main"
tags = ["android"]
title = "MiPlayer 的代码重构"

+++

最近一直忙着进行 MiPlayer 的代码重构，MiPlayer 是我大三的时候做的一个 Android 音乐播放器，开源到 github 上之后得到了几个 star ，也收到一些反馈的问题，自己在测试时也发现了一些比较明显的例如按钮状态同步问题，之前总没有时间去解决，但是心里总有一些愧疚，故而最近将其重新拾起。

改版的方向主要有两个，界面和代码结构。现在看一年前写的代码，发现问题太多了，比如变量命名没有统一的风格，一处是驼峰命名，另一处却没有遵循同样的规范，资源文件的名词之间没有使用下划线进行分割，还有就是代码模块与模块之间耦合性太高等。在界面上我发现原来的图标同样存在着风格不统一的情况，但是短时间做出一套较完善的切图难度很大，所以打算偷懒使用网易云音乐的图标。项目不会放到应用市场上，应该不会有版权等问题，并且我会标注出处。

经过着几天发现代码重构要比推倒重作难的多，慢慢来吧。

![Olive Elise](../../img/about-miplayer/OliveElise.jpg)