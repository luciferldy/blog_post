+++
date = "2015-05-28T16:43:18+08:00"
menu = "main"
tags = ["android"]
title = "Android Service 使用细节"

+++

* 多个 client 可以绑定至一个 service ，但是该service的 `onBind()` 方法只会在第一个 client 绑定至其的时候被调用，当其他 client 再次绑定到它的时候，并不会调用 `onBind()` 方法，而且直接返回第一次调用时产生的那个 IBinder 对象，也就说，在其生命周期内周期内，**onBind()只会被调用一次** 。即使是先 `startService()` 再 `bindService()`，绑定多个客户端，也只调用一次。
* 当 activity 调用 `bindService()` 方法后就会回调 Activity 的 `onServiceConnected`, 在这个方法中向 Activity 中传递一个 IBinder 的实例， Activity 需要保存这个实例。但是我现在看貌似是没有 `onServiceConnected` 这个了，基本都是使用 **ServiceConnection** 这个类进行绑定之后的连接操作的。

2015年5月28日更新

-----------------------------------

* 当使用 bindService 时，其中需要传递的一个参数 ServiceConnection 。需要注意的是，很多情况下是在 ServiceConnection 类的 `onServiceConnected()` 函数中进行 IBinder 的获取以及Service的实例化，比较坑爹的是这个 ServiceConnection 的建立也是一个类似 MediaPlayer 的 `prepare()`过程，所以可能需要对在 `onServiceConnection()` 进行实例化的类监控，否则容易出现 `NullPointerException` 的情况。

2015年5月29日更新
