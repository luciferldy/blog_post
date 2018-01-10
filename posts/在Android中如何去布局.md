+++
date = "2015-05-05T22:40:09+08:00"
menu = "main"
tags = ["android"]
title = "在 Android 中如何去布局"

+++

在学习Android的过程中，关于如何去布局

基础知识
------

首先在Android中我们需要知道的是如下的四个概念，除此之外像英寸(inch)之类的概念可以不用掌握

1. px：像素，也就是屏幕上的点，现在通常来讲手机屏幕分辨率是1920*1080的意思就是宽有1080个像素点，高有1920个像素点。
5. dp：一种基于屏幕密度的抽象单位，在每英寸160点的显示器上 1dp = 1px，在每英寸240点的显示器上 1dp = 1.5px，通常我们在xml文件中对view的宽，高，或者边距使用dp作为单位为其赋值，保证其在1920*1080和1080*720的手机上看起来一样。
6. dip(设备无关像素)：和dp一样，不必深究
7. sp(与刻度无关的像素)：这个是用来给字体作为单位，同样可以做到在不同分辨率设备上显示出同样的效果，我们在布局中也可以使用dip作为字体的单位。

LinearLayout使用技巧
-----------------

linearlayout是线性布局，我们在使用linearlayout时必须为其设置一个叫做 orientation（方向） 的属性，在linearlayout 中还有一个非常好用的属性就是 weight, 这个属性可以将子view的宽高按照weight的值按照比例进行设置。

	<LinearLayout
	        android:layout_width="match_parent"
	        android:layout_height="match_parent"
	        android:orientation="horizontal">
	        <Button
	            android:layout_width="0dp"
	            android:layout_height="wrap_content"
	            android:layout_weight="1"
	            android:text="one"/>
	        <Button
	            android:layout_width="0dp"
	            android:layout_height="wrap_content"
	            android:layout_weight="2"
	            android:text="two"/>
	</LinearLayout>

如上的布局方式能够使 one 和 two 按照 1:2 的比例渲染，为什么要将`layout_width`设置为0呢。当在`LinearLayout`的`child view`中使用`layout_width`时，会`measure`两次，第一次正常计算两个`view`的宽高，第二次将结合`layout_weight`的值分配剩余的空间。当`layout_width`都为0时，保证剩余空间为屏幕宽。

因为，用下面的方式进行布局，得到的结果也就不奇怪了

	<LinearLayout
	        android:layout_width="match_parent"
	        android:layout_height="match_parent"
	        android:orientation="horizontal">
	        <Button
	            android:layout_width="match_parent"
	            android:layout_height="wrap_content"
	            android:layout_weight="1"
	            android:text="one"/>
	        <Button
	            android:layout_width="match_parent"
	            android:layout_height="wrap_content"
	            android:layout_weight="2"
	            android:text="two"/>
	</LinearLayout>

在这种情况下，`one`和`two`的比例是 2:1，为什么会是 2:1 呢？因为根据`layout_weight`的渲染方式，假设屏幕宽度为*L*。

	则剩余宽度 L - 2 * L = -L
	ONE: L + (-L) * 1 / 3 = 2 / 3 L
	TWO: L + (-L) * 2 / 3 = 1 / 3 L

`layout_gravity`和`gravity`的使用技巧
----------------------------------

`layout_gravity`是在布局中使用，控制其内容（也就是子控件）的位置

`gravity`是在控件中使用，控制器内容的位置

在父布局中使用`gravity`和在`child view`中使用`layout_gravity`有时效果一样，但有时有的会无效。


`ScrollView`的使用技巧
-------------------

`ScrollView`继承自`FrameLayout`，所以直接子布局只能有一个。需要注意的是，直接子布局的高度的设置不能和`ScrollView`的高度一致，否则`ScrollView`将不能滑动。