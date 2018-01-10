+++
date = "2015-04-12T19:45:47+08:00"
menu = "main"
tags = ["android"]
title = "Android四种启动模式"

+++

##### Android 启动模式

Android 的四种启动模式分别是 Standard, SingleTop, SingleTask, SingleInstance
 
Android 的任务栈是指 Android 的一系列有关系的 activity 的集合，使用栈封装。通常一个Application 通常就是一个任务栈，但是当任务栈遇到不同的启动方式时，又会有不同的变化。
 
Standard：当A是以 standard 方式启动时，就是正常在原栈里 new 一个 activity 。A->B (start A)=>A->B->A; A->B(start B)=>A->B->B。

SinlgeTop：同样是在原栈里 new 一个 activity N，当栈顶为 N 时，便不会 new N，而是使用原有的 N。A->B(start B)=>A->B; A->B(start A)=>A->B->a。

SingleTask：这种方式启动时，新开一个栈，并且最多只有一个以此方式启动的实例。就如同单例模式中那个单例，所有的操作改动都在这一个里面运行。比较适合构造成本较大，但是切换成本较小的Activity中：比如浏览器的 activity，在QQ，微信，陌陌都要使用本地的浏览器打开一个链接时，基本都只调用那一个实例。

SingleInstance：同 SingleTask 差不多，但是更加极端，新开一个栈，并且栈中只有这一个activity 实例。如果涉及其他的 Activity，需要移交到其他的 Task 中运行。

