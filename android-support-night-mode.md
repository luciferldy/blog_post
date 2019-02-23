+++
date = "2017-02-27T09:14:17+08:00"
menu = "main"
tags = ["android", "code"]
title = "Android 夜间模式切换"

+++



在开发 App 时，为了在夜晚有更好的阅读体验，我们可以为 App 增加夜间模式。下面介绍几种我用过的实现夜间模式的方法


## AppCompatDelegate

AppCompatDelegate 类中有一个方法 `setDefaultNightMode()` ，这个方法可以设置四个值

* MODE\_NIGHT\_NO 日间模式
* MODE\_NIGHT\_YES 夜间模式
* MODE\_NIGHT\_AUTO 自动切换日间夜间模式
* MODE\_NIGHT\_FOLLOW\_SYSTEM 跟随系统

Activity 继承 AppCompatActivity 后可以调用 `AppCompatDelegate.setDefaultNightMode()` 进行夜间模式的切换。 AppCompatDelegate 还有一个方法 `setLocalNightMode()` 用来设置当前 Activity 的夜间模式，在使用 `setLocalNightMode()` 函数时需要对 Activity 进行重启操作，即调用 `recreate()` 。

AppCompatDelegate 配合 Theme.AppCompat.DayNight 主题，在切换日夜间模式时可以显式官方的配色。如果想要自定义资源的话只要在 res 文件夹下创建 values-night 文件夹并创建对应的 themes.xml 文件。

AppCompatDelegate.setDefaultNightMode() 会对 App 的 theme 的 DayNight mode 生效，而 getDelegate().setLocalNightMode() 只会设置当前的 Acitivity ，并且 setLocalNightMode() 会覆盖 DefaultNightMode 的效果。

styles.xml in values

```xml
<!-- Base application theme. -->
<style name="AppThemeDayNight" parent="Theme.AppCompat.DayNight.NoActionBar">
    <!-- Customize your theme here. -->
    <!--   your app branding color for the app bar -->
    <item name="colorPrimary">@color/colorPrimary</item>
    <!--   darker variant for the status bar and contextual app bars -->
    <item name="colorPrimaryDark">@color/colorPrimaryDark</item>
    <!--   theme UI controls like checkboxes and name fields -->
    <item name="colorAccent">@color/colorAccent</item>

</style>
```

colors.xml in vlaues

```xml
<resources>
    <color name="colorPrimary">#00A2ED</color>
    <color name="colorPrimaryDark">#1976D2</color>
    <color name="colorAccent">#FF4081</color>
    <color name="drawerItemSelected">#F0F0F0</color>
    <color name="drawerItemNormal">#FFFFFF</color>
</resources>
```

colors.xml in values-night

```xml
<resources>
    <color name="colorPrimary">#343434</color>
    <color name="colorPrimary">#000000</color>
    <color name="colorAccent">#656565</color>
</resources>
```

在 MainFragment 中当点击某个 item 时

```java
if(item.getTitle().equals(getResources().getString(R.string.mode_night_yes))) {
	getDelegate().setLocalNightMode(AppCompatDelegate.MODE_NIGHT_YES);
	getActivity().recreate();
} else {
	getDelegate().setLocalNightMode(AppCompatDelegate.MODE_NIGHT_NO);
	getActivity().recreate();
}
```

## UiModeManager

UiModeManager 实现夜间模式的方法与 AppCompatDelegate 相似。

```java
UiModeManager uiManager = (UiModeManager) getSystemService(Context.UI_MODE_SERVICE);
if (isNightMode) {
	uiManager.enableCarMode(0);
	uiManager.setNightMode(UiModeManager.MODE_NIGHT_YES);
} else {
	uiManager.disableCarMode(0);
	uiManager.setNightMode(UiModeManager.MODE_NIGHT_NO);
}
```

## 切换 theme

```java
public void changeTheme() {
    if (isNightMode) {
        setTheme(R.style.ThemeDefault);
        isNightMode = false;
    } else {
        setTheme(R.style.ThemeNight);
        isNightMode = true;
    }
    setContentView(R.layout.activity_main);
}
```

## 通过资源 id 映射的方式

通过 id 获取资源时，先将其转化为夜间模式对应的 id ，再通过 Resources 来获取对应的资源

```java
public static Drawable getDrawable(Context context, int id) {
    return context.getResources().getDrawable(getResId(id));
}
public static int getResId(int defaultResId) {
    if (!isNightMode()) {
      return defaultResId;
    }
    if (sResourceMap == null) {
      buildResourceMap();
    }
    int themedResId = sResourceMap.get(defaultResId);
    return themedResId == 0 ? defaultResId : themedResId;
}
```

## 通过修改 uiMode 来切换夜间模式

uiMode 是 Configuration 类的一个变量。首先将获取资源的地方统一起来，使用 Application 对应的 Resources ，在 Application 的 onCreate 中调用 ResourceManager 的 init 方法将其初始化。

```java
public static void init(Context contet) {
	sRes = contenxt.getResources();
}
```

通过更新 uiMode 来更新 Resources 的配置，系统会根据其 uiMode 读取对应 night 下的资源，同时在 res 中给夜间模式的资源添加 -night 后缀，比如 values-night ， drawable-night 。

```java
public static void updateNightMode(boolean on) {
  DisplayMetrics dm = sRes.getDisplayMetrics();
  config = sRes.getConfiguration();
  config.uiMode &= ~Configuration.UI_MODE_NIGHT_MASK;
  config.uiMode |= on ? Configuration.UI_MODE_NIGHT_YES : Configuration.UI_MODE_NIGHT_NO;
  sRes.updateConfiguration(config, dm);
}
```