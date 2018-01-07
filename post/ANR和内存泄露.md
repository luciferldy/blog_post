+++
date = "2015-04-23T08:49:00+08:00"
menu = "main"
tags = ["android"]
title = "ANR 和内存泄露"

+++

### ANR

全称（Application Not Responding）,一般有3种类型  

1. KeyDispatchTimeout(5 seconds) 主要类型是按键或者触摸事件在特定时间内没有响应。
2. BroadcastTimeout(10 seconds) BroadcastReciever在特定时间内没有处理完成。
3. ServiceTimeout(20 seconds) 小概率类型，Service在特定时间内无法处理完成。  

### Android 内存泄漏及分析调试

GC（Garbage Collection）垃圾回收  
GC会选择一些它了解还存活的对象作为内存遍历的根节点（GC roots），比如说Thread, Stack中的变量，JNI中的全局变量，Zygote中的对象（class loader加载）等，然后开始对heap进行遍历到最后，部分没有直接或间接引用到GC roots就是需要回收的垃圾，会被GC 回收掉。  

有哪些情况会导致内存泄漏  

1. 非静态内部类的静态实例容易造成内存泄漏。
2. activity使用静态成员（listview优化时将viewholder变成静态内部类好像也能造成内存泄漏）
3. 使用handler时存在问题，handler通过发送Message与主线程进行互动，Message Queue中存在很多Message，这些Message可能不会被马上处理掉。Message在Queue中时间越长，handler便无法被回收。
4. 注册某个对象时没有反注册。