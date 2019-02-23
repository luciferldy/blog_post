+++
date = "2015-12-01T15:50:18+08:00"
menu = "main"
tags = ["android"]
title = "Android中click,touch,focus之间的关系"

+++

今天想说一下Android中关于click,touch,focus之间的关系.

首先在官网上View的`setFocusable(boolean focusable)`中如下解释

	set whether this view can recieve the focus, setting this to false will also ensure that this vie is not focusable in touch mode.

下面进行我的一些测试

#### 正常
content_main.xml

	<?xml version="1.0" encoding="utf-8"?>
	<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
		xmlns:tools="http://schemas.android.com/tools"
		xmlns:app="http://schemas.android.com/apk/res-auto"
		android:id="@+id/touch_container"
		android:layout_width="match_parent"
		android:layout_height="match_parent"
		android:paddingLeft="@dimen/activity_horizontal_margin"
		android:paddingRight="@dimen/activity_horizontal_margin"
		android:paddingTop="@dimen/activity_vertical_margin"
		android:paddingBottom="@dimen/activity_vertical_margin"
		app:layout_behavior="@string/appbar_scrolling_view_behavior"
		tools:showIn="@layout/activity_main" 
		tools:context=".MainActivity">
	
		<Button
		  android:id="@+id/touch_me"
		  android:layout_width="wrap_content"
		  android:layout_height="wrap_content"
		  android:text="touch me like you do"/>
	</RelativeLayout>

MainActivity.java

	touchMe = (Button) findViewById(R.id.touch_me);
    touchMe.setOnClickListener(new View.OnClickListener() {
        @Override
        public void onClick(View view) {
            TAG = BASE_TAG + " button";
            Log.d(TAG, "onClick");
        }
    });
    touchMe.setOnTouchListener(new View.OnTouchListener() {
        @Override
        public boolean onTouch(View view, MotionEvent motionEvent) {
            TAG = BASE_TAG + " button";
            Log.d(TAG, "onTouch");
            switch (motionEvent.getAction()) {
                case MotionEvent.ACTION_DOWN:
                    Log.d(TAG, "action down");
                    break;
                case MotionEvent.ACTION_UP:
                    Log.d(TAG, "action up");
                    break;
                case MotionEvent.ACTION_MOVE:
                    Log.d(TAG, "action move");
                    break;
                default:
                    Log.d(TAG, "action others");
                    break;
            }
            return false;
        }
    });
    touchCon = (RelativeLayout) findViewById(R.id.touch_container);
    touchCon.setOnClickListener(new View.OnClickListener() {
        @Override
        public void onClick(View view) {
            TAG = BASE_TAG + " relativelayout";
            Log.d(TAG, "onClick");
        }
    });
    touchCon.setOnTouchListener(new View.OnTouchListener() {
        @Override
        public boolean onTouch(View view, MotionEvent motionEvent) {
            TAG = BASE_TAG + " relativelayout";
            Log.d(TAG, "onTouch");
            switch (motionEvent.getAction()) {
                case MotionEvent.ACTION_DOWN:
                    Log.d(TAG, "action down");
                    break;
                case MotionEvent.ACTION_UP:
                    Log.d(TAG, "action up");
                    break;
                case MotionEvent.ACTION_MOVE:
                    Log.d(TAG, "action move");
                    break;
                default:
                    break;
            }
            return false;
        }
    });

    点击除了按钮之外的情况
    12-01 16:10:33.235 31970-31970/com.lucifer D/MainActivity relativelayout: onTouch
    12-01 16:10:33.235 31970-31970/com.lucifer D/MainActivity relativelayout: action down
    12-01 16:10:33.275 31970-31970/com.lucifer D/MainActivity relativelayout: onTouch
    12-01 16:10:33.275 31970-31970/com.lucifer D/MainActivity relativelayout: action up
    12-01 16:10:33.275 31970-31970/com.lucifer D/MainActivity relativelayout: onClick

    点击按钮
    12-01 16:11:13.490 307-307/com.lucifer D/MainActivity button: onTouch
    12-01 16:11:13.490 307-307/com.lucifer D/MainActivity button: action down
    12-01 16:11:13.500 307-307/com.lucifer D/MainActivity button: onTouch
    12-01 16:11:13.500 307-307/com.lucifer D/MainActivity button: action up
    12-01 16:11:13.515 307-307/com.lucifer D/MainActivity button: onClick

当触摸按钮并从划出按钮区域时,不会触发relativelayout的touch事件

#### RelativeLayout将focusable设置为false

点击除了按钮之外的情况

	12-01 16:15:11.935 1885-1885/com.lucifer D/MainActivity relativelayout: onTouch
	12-01 16:15:11.935 1885-1885/com.lucifer D/MainActivity relativelayout: action down
	12-01 16:15:11.965 1885-1885/com.lucifer D/MainActivity relativelayout: onTouch
	12-01 16:15:11.965 1885-1885/com.lucifer D/MainActivity relativelayout: action up
	12-01 16:15:11.965 1885-1885/com.lucifer D/MainActivity relativelayout: onClick

点击按钮

	12-01 16:16:46.595 3046-3046/com.lucifer D/MainActivity button: onTouch
	12-01 16:16:46.595 3046-3046/com.lucifer D/MainActivity button: action down
	12-01 16:16:46.630 3046-3046/com.lucifer D/MainActivity button: onTouch
	12-01 16:16:46.630 3046-3046/com.lucifer D/MainActivity button: action up
	12-01 16:16:46.645 3046-3046/com.lucifer D/MainActivity button: onClick

#### RelativeLayout将focusableInTouchMode设置为false

情况如上

#### RelativeLayout将clickable设置为false

RelativeLayout还是能够响应点击事件。我分析, 因为对于content_main.xml的渲染是在代码执行之前,所以即使设置了clickable为false,但是后面setOnClickListener的时候,可能将clickable设置为了true。所以我在setOnClickListener()后面添加了setClickable(false),打印出的日志如下

	12-01 16:31:55.410 12627-12627/com.lucifer D/MainActivity relativelayout: onTouch
	12-01 16:31:55.415 12627-12627/com.lucifer D/MainActivity relativelayout: action down

RelativeLayout无法响应click事件

但是此时点击Button

	12-01 16:31:52.415 12627-12627/com.lucifer D/MainActivity button: onTouch
	12-01 16:31:52.415 12627-12627/com.lucifer D/MainActivity button: action down
	12-01 16:31:52.460 12627-12627/com.lucifer D/MainActivity button: onTouch
	12-01 16:31:52.460 12627-12627/com.lucifer D/MainActivity button: action up
	12-01 16:31:52.485 12627-12627/com.lucifer D/MainActivity button: onClick

Button可以响应touch和click事件


#### Button将focusable设置为false

情况如上,看来focusable没什么卵用。

#### Button将clickable设置为false

Button还是能够响应点击事件的，我假设,因为对于content_main.xml的渲染是在代码执行之前,所以即使设置了clickable为false,但是后面setOnClickListener的时候,可能将clickable设置为了true。对此我做了以下实验：
我在setOnClickListener()后面添加了setClickable(false),打印出的日志如下

	12-01 16:28:08.390 11542-11542/com.lucifer D/MainActivity button: onTouch
	12-01 16:28:08.390 11542-11542/com.lucifer D/MainActivity button: action down
	12-01 16:28:08.390 11542-11542/com.lucifer D/MainActivity relativelayout: onTouch
	12-01 16:28:08.390 11542-11542/com.lucifer D/MainActivity relativelayout: action down
	12-01 16:28:08.435 11542-11542/com.lucifer D/MainActivity relativelayout: onTouch
	12-01 16:28:08.435 11542-11542/com.lucifer D/MainActivity relativelayout: action up
	12-01 16:28:08.445 11542-11542/com.lucifer D/MainActivity relativelayout: onClick

ok,过~

#### Button将enable设置为false

点击Button就不能用了
