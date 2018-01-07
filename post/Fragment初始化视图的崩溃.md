+++
date = "2016-01-11T11:33:09+08:00"
menu = "main"
tags = ["android"]
title = "Fragment初始化视图的崩溃"

+++

这两天在做一个RSS应用时, 我在Activity中添加Fragment时,遇到了一个崩溃

	01-10 22:47:19.794 4772-4772/com.gank.io E/AndroidRuntime: FATAL EXCEPTION: main
	Process: com.gank.io, PID: 4772
	java.lang.IllegalStateException: The specified child already has a parent. You must call removeView() on the child's parent first.
	at android.view.ViewGroup.addViewInner(ViewGroup.java:3562)
	at android.view.ViewGroup.addView(ViewGroup.java:3415)
	at android.view.ViewGroup.addView(ViewGroup.java:3360)
	at android.view.ViewGroup.addView(ViewGroup.java:3336)
	at android.support.v4.app.FragmentManagerImpl.moveToState(FragmentManager.java:1083)
	at android.support.v4.app.FragmentManagerImpl.moveToState(FragmentManager.java:1248)
	at android.support.v4.app.BackStackRecord.run(BackStackRecord.java:738)
	at android.support.v4.app.FragmentManagerImpl.execPendingActions(FragmentManager.java:1613)
	at android.support.v4.app.FragmentManagerImpl$1.run(FragmentManager.java:517)
	at android.os.Handler.handleCallback(Handler.java:733)
	at android.os.Handler.dispatchMessage(Handler.java:95)
	at android.os.Looper.loop(Looper.java:136)
	at android.app.ActivityThread.main(ActivityThread.java:5001)
	at java.lang.reflect.Method.invokeNative(Native Method)
	at java.lang.reflect.Method.invoke(Method.java:515)
	at com.android.internal.os.ZygoteInit$MethodAndArgsCaller.run(ZygoteInit.java:785)
	at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:601)
	at dalvik.system.NativeStart.main(Native Method)

说实话, 这个崩溃的意思就是获取一个View却绕过了它的Parent,我的布局是这样的

	<?xml version="1.0" encoding="utf-8"?>
	<android.support.v4.widget.NestedScrollView
		xmlns:android="http://schemas.android.com/apk/res/android"
		xmlns:fresco="http://schemas.android.com/apk/res-auto"
		android:id="@+id/content_meizhi_root"
		android:orientation="vertical"
		android:layout_width="match_parent"
		android:layout_height="match_parent"
		android:background="@android:color/white">
		
		<LinearLayout
		  android:id="@+id/content_meizhi_rss"
		  android:layout_width="match_parent"
		  android:layout_height="wrap_content"
		  android:orientation="vertical">
		
		  <com.facebook.drawee.view.SimpleDraweeView
		      android:id="@+id/content_meizhi_img"
		      android:layout_width="match_parent"
		      android:layout_height="300dip"
		      fresco:placeholderImage="@mipmap/ic_launcher"/>
		
		</LinearLayout>
	
	</android.support.v4.widget.NestedScrollView>

我的代码是这样的

	@Nullable
	@Override
	public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
	 View root = inflater.inflate(R.layout.news_content, container);
	 rssContent = (LinearLayout) root.findViewById(R.id.content_meizhi_rss);
	 Bundle bundle = getArguments();
	 String year, month, day;
	 if (bundle != null) {
	     year = bundle.getString("year");
	     month = bundle.getString("month");
	     day = bundle.getString("day");
	     new LoadContentTask().execute(year + "/" + month + "/" + day);
	 }
	 return root;
	}

但是对于产生崩溃的原因我却百思不得其解, 因为时渲染视图的问题,所以我先后注释掉了

	meizhiimg = (SimpleDraweeView) root.findViewById(R.id.meizhi_preview_img);

以及以后的内容, 崩溃依然存在, 后来我发现

	View root = inflater.inflate(R.layout.news_content, container);

改成

	View root = inflater.inflate(R.layout.news_content, container, false);

后崩溃消失, 后来我去官方文档上看了一下这两个函数的区别

	public View inflate (int resource, ViewGroup root)
	Inflate a new view hierarchy from the specified xml resource. Throws InflateException if there is an error. 
	Returns: The root View of the inflated hierarchy. If root was supplied, this is the root View; otherwise it is the root of the inflated XML file.

这个函数的返回值说的是当提供了Fragment的根布局的话就返回根部句,当然我在Activity中是这样添加的

	FragmentTransaction transaction = manager.beginTransaction();
	transaction.add(android.R.id.content, newsItem);
	transaction.addToBackStack(null);
	transaction.commit();

后来我在Fragment的`onCreateView`中添加了如下一句

	if (root.getId() == android.R.id.content) {
	  Log.d(LOG_TAG, "root.getId() == android.R.id.content");
	}

Log证实了确实是相等的, 因此我在 `onCreateView` 的返回值是 `android.R.id.content` 而这个返回值会造成 InflateException 的异常, 具体的原因我会在下一篇博客解释. 建议大家使用下面的函数

	public View inflate (int resource, ViewGroup root, boolean attachToRoot)
	Inflate a new view hierarchy from the specified xml resource.

对于attachToRoot的解释

	attachToRoot
	Whether the inflated hierarchy should be attached to the root parameter? If false, root is only used to create the correct subclass of LayoutParams for the root view in the XML.

	Throws InflateException if there is an error. Returns: The root View of the inflated hierarchy. If root was supplied and attachToRoot is true, this is root; otherwise it is the root of the inflated XML file.

并且将attachToRoot设置为false, 这样 inflate() 的返回值就是渲染的xml的根了.
