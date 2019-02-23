+++
date = "2017-02-27T09:13:41+08:00"
menu = "main"
tags = ["android", "code"]
title = "状态栏与沉浸式分析"

+++



在 Android 4.4 SDK(API Level 19) 中， Google 为设置系统 UI 的可见性引入了一个新的 flag: SYSTEM_UI_FLAG_IMMERSIVE 。当这个 flag 与 SYSTEM_UI_FLAG_HIDE_NAVIGATION 和 SYSTEM_UI_FLAG_FULL_SCREEN flag 一起使用时，可以隐藏导航栏和状态栏并可以让你的 app 能够捕获全屏的 touch 事件。（note: "immersive" flag 只有与 SYSTEM_UI_FLAG_HIDE_NAVIGATION 或 SYSTEM_UI_FLAG_FULL_SCREEN 一起使用时才会生效）

## 全屏模式

SYSTEM_UI_FLAG_IMMERSIVE 实现的就是沉浸式，但是国内貌似把沉浸式这种全屏模式与状态栏一体化的概念混淆了。全屏共有两种样式。

> Lean back: touch the screen anywhere to bring back system bars.
>
> Immersive: swipe from any edge of the screen with a hidden bar to bring back system bars.

Lean back

![Lean back](../../img/status_bar_and_immersive_mode/fullscreen_leanback.png)

Immersive

![Immersive](../../img/status_bar_and_immersive_mode/fullscreen_immersive_swipe_top.png)

在 Google Android 官方文档中提到如何使用上面的几个标签：

* 如果是想做一个 book reader, news reader, or magazine 。使用 IMMERSIVE , SYSTEM_UI_FLAG_FULLSCREEN ( SYSTEM_UI_FLAG_HIDE_NAVIGATION ) ，因为用户需要经常接触到 actionbar 或者其他 ui 元素。
* 如果你想要做一个真正的沉浸式应用程序，不要用户经常操作系统 ui ，使用 IMMERSIVE, SYSTEM_UI_FLAG_FULLSCREEN ( SYSTEM_UI_FLAG_HIDE_NAVIGATION) ，这个方案比较适合游戏或者画图程序。
* 如果想做一个视频播放器的话，需要减少用户的动画，使用 Lean back 就好，也就是简单的使用 SYSTEM_UI_FLAG_FULLSCREEN 和 SYSTEM_UI_FLAG_HIDE_NAVIGATION 就已经足够，不要使用 "immersive" 标签。



在程序切换至全屏模式时，应用的 content 会随着 status bar 和 navigation bar 的隐藏而重新计算，为避免这种抖动，我们可以使用 SYSTEM_UI_FLAG_LAYOUT_HIDE_NAVIGATION 和  SYSTEM_UI_FLAG_LAYOUT_STABLE 。

```java
// This snippet hides the system bars.
private void hideSystemUI() {
    // Set the IMMERSIVE flag.
    // Set the content to appear under the system bars so that the content
    // doesn't resize when the system bars hide and show.
    mDecorView.setSystemUiVisibility(
            View.SYSTEM_UI_FLAG_LAYOUT_STABLE
            | View.SYSTEM_UI_FLAG_LAYOUT_HIDE_NAVIGATION
            | View.SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN
            | View.SYSTEM_UI_FLAG_HIDE_NAVIGATION // hide nav bar
            | View.SYSTEM_UI_FLAG_FULLSCREEN // hide status bar
            | View.SYSTEM_UI_FLAG_IMMERSIVE);
}

// This snippet shows the system bars. It does this by removing all the flags
// except for the ones that make the content appear under the system bars.
private void showSystemUI() {
    mDecorView.setSystemUiVisibility(
            View.SYSTEM_UI_FLAG_LAYOUT_STABLE
            | View.SYSTEM_UI_FLAG_LAYOUT_HIDE_NAVIGATION
            | View.SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN);
}
```

`setSystemUiVisibility()` 和`getSystemUiVisibility()` 方法，实现对状态栏的动态显示或隐藏，以及获取状态栏目前的可见性。

`setSystemUiVisibility(int visibility)` 方法可传入的参数为

> SYSTEM_UI_FLAG_VISIBLE  显式状态栏，activity 不全屏显示
>
> SYSTEM_UI_FLAG_INVISIBLE  隐藏状态栏，同时 activity 会伸展到全屏显示
>
> SYSTEM_UI_FLAG_FULLSCREEN  activity 全屏显示，状态栏隐藏
>
> SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN  activity 全屏显示，但是状态栏可见，activity 顶端布局会被状态栏遮挡
>
> SYSTEM_UI_FLAG_LAYOUT_HIDE_NAVIGATION  效果同上
>
> SYSTEM_UI_FLAG_LAYOUT_FLASS  效果同 SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN
>
> SYSTEM_UI_FLAG_HIDE_NAVIGATION  隐藏导航栏
>
> SYSTEM_UI_FLAG_LAYOUT_LOW_PROFILE 状态栏显示处于低能显示状态，状态栏上的一些图标会被隐藏



sticky 标签不会触发任何的监听器，因为在这个模式下显示的系统 ui 是暂时的状态。

在 4.0 以下的版本可以通过使用 `theme="@android:style/Theme.Holo.NoActionBar.FullScreen"` 和

```java
public class MainActivity extends Activity {
  @Override
  protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    // If the Android version is lower than Jellybean, use this to hide
    // the status bar.
    if (Build.VERSION.SDK_INT < 16) {
      getWindow().setFlags(WindowManager.LayoutParams.FLAG_FULLSCREEN, WindowManager.LayoutParams.FLAG_FULLSCREEN);
    }
    setContent(R.layout.activity_main);
  }
}
```

来实现，当你设置了 WindowManager 的flag 时，将一直保持 flag 的效果，除非你重置了 flag ，但当你调用了 setSystemUiVisibility() 函数设置 immersive 状态时，swipe from edge 的操作会清除 SYSTEM_UI_FLAG_HIDE_NAVIGATION 等 flag 。(SYSTEM_UI_FLAG_LAYOUT_HIDE_NAVIGATION 等布局的 flag 不会被清除掉)

官方中写道：为了防止 status bar 切换隐藏和显示状态是 activity 界面会出现重新分配 ui 的情况，可以使用 FLAG_LAYOUT_IN_SCREEN 。（官方是这样说，但是还是会有一些问题，当只是普及了）

界面的切换也会导致 setSystemUiVisibility() 的设置被清空。

在 Android 4.1 以及之后的版本，可以让 activity 的内容部分显示在 status bar 的背后，status bar 不再影响到实际内容的空间布局（以前内容总在 status bar 下面）。

将 activity 的内容区域放在 status bar 后面，需要确保被挡住的部分不进行操作，大多数情况下在布局文件的 root elements 添加 `android:fitSystemWindows` 属性。可以解决 status bar 挡住内容区域的问题，因为改设置可以调整 viewGroup 的 topPadding ，为系统 ui 预留出一定的区域。

对于 fitSystemWindows 属性，官方的解释为

> Boolean internal attribute to adjust view layout based on system windows such as the status bar. If true, adjusts the padding of this view to leave space for the system windows. Will only take effect if this view is in a non-embedded activity



## 状态栏一体化

状态栏一体化就是把内容布局扩展到状态栏，并将状态栏的颜色设置为透明或者与主题相近的颜色。

### 使用 fitSystemWindows 属性

上文中说过，`android:fitSystemWindows` 属性就是在 view 增加一个 topPadding 。于是我们可以把 content 扩展到 fullscreen 并在布局的 root element 上添加 `android:fitSystemWindows="true"` 就好。代码如下

```java
Window window = getWindow();
// < 4.1 可能需要搭配 theme="@android:style/Theme.Holo.NoActionBar.FullScreen" 使用
if (Build.VERSION.SDK_INT < Build.VERSION_CODES.JELLY_BEAN) {
  window.setFlags(WindowManager.LayoutParams.FLAG_FULLSCREEN, WindowManager.LayoutParams.FLAG_FULLSCREEN);
}

// >= 4.1
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.JELLY_BEAN) {
  window.clearFlags(WindowManager.LayoutParams.FLAG_FULLSCREEN);
  window.getDecorView().setSystemUiVisibility(View.SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN | View.SYSTEM_UI_FLAG_LAYOUT_STABLE);
}

// 4.4 translucent
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.KITKAT) {
  getWindow().addFlags(WindowManager.LayoutParams.FLAG_TRANSLUCENT_STATUS);
}

// 5.0 transparent
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {
  window.clearFlags(WindowManager.LayoutParams.FLAG_TRANSLUCENT_STATUS);
  window.addFlags(WindowManager.LayoutParams.FLAG_DRAWS_SYSTEM_BAR_BACKGROUNDS);
  window.setStatusBarColor(Color.TRANSPARENT);
}
```

其实上述代码也可以在 styles.xml 实现

styles.xml in values-v19 

```xml
<style name="AppTheme.NoActionBar">
	<item name="android:windowTranslucentStatus">true</item>
</style>
```

styles.xml in values-v21

```xml
<style name="AppTheme.NoActionBar">
  <item name="android:windowDrawsSystemBarBackgrounds">true</item>
  <item name="android:statusBarColor">@android:color/transparent</item>
</style>
```

statusbar 处显示的颜色在 colors.xml 中 colorPrimaryDark 属性设置





### 自定义占位 status bar

这种方案不使用系统的 `android:fitSystemWindows="true"` 属性，而是自己添加一个 view 用来占据 content 扩展至全屏后被 status bar 遮挡的部分。

model_toolbar.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<android.support.design.widget.AppBarLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:id="@+id/toolbar_layout"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    app:theme="@style/AppTheme.AppBarOverlay">

    <View
        android:id="@+id/status_bar_holder"
        android:layout_width="match_parent"
        android:layout_height="25dp"
        android:background="@color/colorPrimaryDark"
        android:visibility="gone"/>

    <android.support.v7.widget.Toolbar
        android:id="@+id/toolbar"
        android:layout_width="match_parent"
        android:layout_height="?attr/actionBarSize"
        android:background="?attr/colorPrimary"
        app:layout_scrollFlags="scroll|enterAlways"
        app:popupTheme="@style/AppTheme.PopupOverlay" />

</android.support.design.widget.AppBarLayout>
```

activity_main.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<android.support.design.widget.CoordinatorLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="@color/md_white"
    tools:context=".ui.activity.MainActivity">

    <include layout="@layout/model_toolbar" />

    <include layout="@layout/content_main" />

    <android.support.design.widget.FloatingActionButton
        android:id="@+id/fab"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="bottom|end"
        android:layout_margin="@dimen/fab_margin"
        android:src="@drawable/github"
        android:scaleType="centerCrop"/>

</android.support.design.widget.CoordinatorLayout>
```

代码中关于 status bar 状态的设置和第一部分相同，不同的是在 MainActivity 的 `onCreate()` 中

```java
setContentView(R.layout.activity_main);
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.KITKAT) {
  View view = findViewById(R.id.status_bar_holder);
  AppBarLayout.LayoutParams params = (AppBarLayout.LayoutParams) view.getLayoutParams();
  params.height = CommonUtils.getStatusbarHeight(getBaseContext());
  view.setLayoutParams(params);
  view.setVisibility(View.VISIBLE);
}
```

关于 `getStatusbarHeight()`

```java
/**
  * get the statusbar height
  * @param context
  * @return
*/
public static int getStatusbarHeight(Context context) {
  if (statusBarHeight == 0) {
    try {
      Class<?> c = Class.forName("com.android.internal.R$dimen");
      Object o = c.newInstance();
      Field field = c.getField("status_bar_height");
      int x = (Integer) field.get(o);
      statusBarHeight = context.getResources().getDimensionPixelSize(x);
    } catch (Exception e) {
    	e.printStackTrace();
    }
  }
  return statusBarHeight;
}
```

ok，done！