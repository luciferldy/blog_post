+++
date = "2017-03-05T20:31:25+08:00"
menu = "main"
tags = ["android", "bitmap"]
title = "Android 中 Bitmap 占用多少的内存"

+++



## 前言

在面试的时候经常会被问道这样的问题，看了这篇文章之后 [Android 开发绕不过的坑：你的 Bitmap 究竟占多大内存？](http://bugly.qq.com/bbs/forum.php?mod=viewthread&tid=498) 把其中所讲概括一下。

首先在开发过程中我们需要对许多机型和屏幕做适配，我们用到的单位就是 dip 或 dp 或 sp 。实习时，设计师给出 1920*1080 机型上的切图和设计图纸，通常都会把切图放在 xxhdpi 的文件夹下，定义元素的宽高时将图纸上的标注的 pixel 除以 3 然后使用 dip 作为单位。之所以这样做，是因为 1920\*1080 像素的屏幕上 density = 3 。density 是 DisplayMetrics 类的一个成员，在源码中的注释是这样的

```java
/**
 * The logical density of the display.  This is a scaling factor for the
 * Density Independent Pixel unit, where one DIP is one pixel on an
 * approximately 160 dpi screen (for example a 240x320, 1.5"x2" screen), 
 * providing the baseline of the system's display. Thus on a 160dpi screen 
 * this density value will be 1; on a 120 dpi screen it would be .75; etc.
 *  
 * <p>This value does not exactly follow the real screen size (as given by 
 * {@link #xdpi} and {@link #ydpi}, but rather is used to scale the size of
 * the overall UI in steps based on gross changes in the display dpi.  For 
 * example, a 240x320 screen will have a density of 1 even if its width is 
 * 1.8", 1.3", etc. However, if the screen resolution is increased to 
 * 320x480 but the screen size remained 1.5"x2" then the density would be 
 * increased (probably to 1.5).
 *
 * @see #DENSITY_DEFAULT
 */
public float density;

/**
* The screen density expressed as dots-per-inch.  May be either
* {@link #DENSITY_LOW}, {@link #DENSITY_MEDIUM}, or {@link #DENSITY_HIGH}.
*/
public int densityDpi;
```

也就是说在 160dpi 的设备上，1dip = 1px 。而 xxhdpi 对应的是 480dpi ，这样就不难理解为啥要除以 3 了。



## 计算 Bitmap 内存占用

决定一张图片占用内存的因素主要有三点

1. 色彩格式，这个在 Bitmap 的 Config 枚举类中有说明，ARGB_8888 占用 4 个字节。
2. 原始文件存放的资源目录，例如 hdpi ，xhdpi 和 xxhdpi 等。
3. 目标屏幕的密度。

我们知道显示一张图片需要从 .png 或者 .jpg 的原图中 decode 出 bitmap ，我们需要计算 bitmap 的宽高，bitmap 的宽高与图片存放的位置的 dpi 和屏幕的实际 dpi 有关，当原始资源的 density 与目标屏幕的 targetDensity 不一致时，对 bitmap 会有缩放的处理，缩放的比例为 targetDensity * density 。例如将一张 512*512 的图片放在 xxhdpi 文件夹下，在一个分辨率为 1920\*1080 的手机使用 ImageView 展示，大小应该为 (512\*480/480) \* (512\*480/480) * 4 = 1048576B 。在代码中测试一下

```java
ImageView iv = (ImageView) findViewById(R.id.image);
Drawable drawable = iv.getDrawable();
if (drawable instanceof BitmapDrawable) {
  Bitmap b = ((BitmapDrawable) drawable).getBitmap();
  Log.d("ImageActivity", "bitmap drawable size = " + b.getByteCount());
}
Bitmap bitmap = BitmapFactory.decodeResource(getResources(), R.drawable.biabiabia);
Log.d("ImageActivity", "bitmap size = " + bitmap.getByteCount());
```

结果为

```
D/ImageActivity: bitmap drawable size = 1048576
D/ImageActivity: bitmap size = 1048576
```



## 减少 Bitmap 内存占用

### 合理使用 jpg 和 png

jpg 是有损压缩的图片存储格式，无 alpha 通道，png 是无损压缩的图片存储格式。

### 使用 inSampleSize 

在使用 BitmapFactory 的  `decodeResouce()`  时可以设置采样率，如果设置为2，可以把 bitmap 的值减小为原来的 1/4 。

```java
BitmapFactory.Options op = new BitmapFactory.Options();
op.inSampleSize = 2;
Bitmap bm = BitmapFactory.decodeResource(getResources(), R.drawable.biabiabia, op);
Log.d("ImageActivity", "sample 2 bitmap size = " + bm.getByteCount());
```

### 选择合理的 Bitmap 像素格式

前面说的几种 Bitmap 的像素格式都在 Bitmap.Config 类中有介绍

* ALPHA_8  每个像素存储为一个 alpha 通道。
* RGB_555  每个像素占用 2 个字节，只有 RGB 通道才能解析，红色和蓝色占 5 bit ，绿色占 6 bit 。
* ARGB_4444  已经被弃
* ARGB_8888  每个像素占 4 个字节，每个通道占 8 bit 。

ARGB_8888 是最常用的像素格式，如果图片不需要 alpha 通道，特别资源本身为 .jpg 格式时，简易使用 RGB\_555 格式。