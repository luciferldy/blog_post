+++
date = "2015-10-03T21:41:06+08:00"
menu = "main"
tags = ["android"]
title = "再次遇到Fragment"

+++

之前写过一篇关于 Fragment 的文章，然后最近在开发过程中又遇到了一些问题，现在拿到这里来分享一下。

为了探究 Activity 和 Fragment 之间的关系，我写了两个类 MainActivity 和 MainFragment， 并且在 MainActivity 的 `onCreate()` 函数中添加 Fragment， 然后将 Activity 和 Fragment 的生命周期中需要执行的函数分别打印出来，其执行顺序如下。

	MainActivity:onCreate()
	MainActivity:onStart()
	MainFragment:onAttach()
	MainFragment:onCreate()
	MainFragment:onCreateView()
	MainFragmnet:onStart()
	MainActivity:onResume()
	MainFragment:onResume()
	MainActivity:onCreateOptionMenu()
	
	息屏后
	
	MainActivity:onPause()
	MainFragment:onPasue()
	MainActivity:onStop()
	MainFragment:onStop()
	
	点亮屏幕
	
	MainActivity:onStart()
	MainFragment:onStart()
	MainActivity:onResume()
	MainFramgent:onResume()
	
	退出 Fragment
	
	MainFragment:onPasue()
	MainFragment:onStop()
	MainFragment:onDestroy()
	MainFragment:onDestroyView()
	MainFragment:onDettach()

由上述可见，fragment 更像是 activity 的内容，fragment 的存在和交互均依赖于 activity，当 activity 中的 fragment 销毁时并不会调用 activity 的 `onResume()`。 同样，当 acitivity 中有新的 fragment 添加时， activity 中的也不会有反应。

我们通常会在 Activity 中使用 `fragment.addToBackStack(null)` 将 fragment 压入回退栈， 我们可以在 fragment 中使用 `getSupportFragmentManager.popBackStack()` 将压入的 fragment 弹出。

------------------------------------

PS：在开发过程中我们使用 Fragment 有两种， 一种是 app 包里的，另一种是 v4 包里的，v4 包的 Fragment 是为了向下兼容低版本的 API。 使用方式如下：

	v4 包
	
	FragmentManager fm = getSupportFragmentManager();
	
	app 包
	
	FragmentManager fm = getFragmentManager();
