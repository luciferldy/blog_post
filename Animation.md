+++
date = "2015-04-23T12:30:33+08:00"
menu = "main"
tags = ["android"]
title = "关于 Android 动画"

+++

#### Animation 

##### 介绍  

Animation 是一个实现 Android UI 界面动画效果的 API, Animations 提供了一系列的动画效果，可以进行旋转，缩放，淡入淡出等。这些效果可以应用在绝大数的控件中。  

##### 分类  

1. Tweened Animations: 该类 Animations 提供了旋转，移动，伸展和淡出效果。 
	* Alpha 淡入淡出
	* Scale 缩放
	* Rotate 旋转
	* Translate 移动
2. Frame-by-Frame Animation: 这一类 Animation 可以创建一个 Drawable 序列，这些 Drawable 可以按照指定的时间间隔一个一个的显示。  

##### 使用方法（代码中使用）

`Animation extends Object implements Cloneable`

使用 Tweened Animation 的步骤：  

1. 创建一个 AnimationSet 对象（ Animation 子类）
2. 增加需要创建相应的 Animation 对象
3. 为 Animation 对象设置相应的数据
4. 将 Animation 对象添加到 AnimationSet 对象当中
5. 使用控件对象开始执行 AnimationSet  

Tweened Animations 分类  

1. Alpha: 淡入淡出
2. Scale: 缩放
3. Rotate: 旋转
4. Translate: 移动

Animation的四个子类：  
AlphaAnimation, TranslateAnimation, ScaleAnimation, RotateAnimation  

##### 具体实现  

activity_main.xml  

	<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
	    xmlns:tools="http://schemas.android.com/tools"
	    android:layout_width="match_parent"
	    android:layout_height="match_parent"
	    android:paddingLeft="@dimen/activity_horizontal_margin"
	    android:paddingRight="@dimen/activity_horizontal_margin"
	    android:paddingTop="@dimen/activity_vertical_margin"
	    android:paddingBottom="@dimen/activity_vertical_margin"
	    tools:context=".MainActivity"
	    android:orientation="vertical">
	
	    <LinearLayout
	        android:layout_width="wrap_content"
	        android:layout_height="wrap_content"
	        android:orientation="horizontal">
	        <TextView
	            android:id="@+id/rotate_btn"
	            android:layout_width="wrap_content"
	            android:layout_height="wrap_content"
	            android:text="旋转"/>
	        <TextView
	            android:id="@+id/scale_btn"
	            android:layout_width="wrap_content"
	            android:layout_height="wrap_content"
	            android:layout_marginLeft="5dp"
	            android:text="缩放"/>
	        <TextView
	            android:id="@+id/alpha_btn"
	            android:layout_width="wrap_content"
	            android:layout_height="wrap_content"
	            android:layout_marginLeft="5dp"
	            android:text="淡入淡出"/>
	        <TextView
	            android:id="@+id/translate_btn"
	            android:layout_width="wrap_content"
	            android:layout_height="wrap_content"
	            android:layout_marginLeft="5dp"
	            android:text="移动"/>
	    </LinearLayout>
	
	    <LinearLayout
	        android:layout_width="wrap_content"
	        android:layout_height="wrap_content"
	        android:orientation="vertical"
	        android:gravity="center">
	    
	        <ImageButton
	            android:id="@+id/image"
	            android:layout_width="wrap_content"
	            android:layout_height="wrap_content"
	            android:src="@drawable/qi"/>
	    </LinearLayout>
	
	</LinearLayout>

MainActivity.java  

	package com.main.maybe.animationtest;
	
	import android.os.Bundle;
	import android.support.v7.app.ActionBarActivity;
	import android.view.Menu;
	import android.view.MenuItem;
	import android.view.View;
	import android.view.animation.AlphaAnimation;
	import android.view.animation.Animation;
	import android.view.animation.AnimationSet;
	import android.view.animation.RotateAnimation;
	import android.view.animation.ScaleAnimation;
	import android.view.animation.TranslateAnimation;
	import android.widget.ImageView;
	import android.widget.TextView;
	
	
	public class MainActivity extends ActionBarActivity {
	
	    private TextView rotate_btn = null;
	    private TextView scale_btn = null;
	    private TextView translate_btn = null;
	    private TextView alpha_btn = null;
	    private ImageView image = null;
	
	    @Override
	    protected void onCreate(Bundle savedInstanceState) {
	        super.onCreate(savedInstanceState);
	        setContentView(R.layout.activity_main);
	
	        rotate_btn = (TextView) findViewById(R.id.rotate_btn);
	        scale_btn = (TextView) findViewById(R.id.scale_btn);
	        translate_btn = (TextView) findViewById(R.id.translate_btn);
	        alpha_btn = (TextView) findViewById(R.id.alpha_btn);
	        image = (ImageView) findViewById(R.id.image);
	
	        rotate_btn.setOnClickListener(new View.OnClickListener() {
	            @Override
	            public void onClick(View v) {
	
	                AnimationSet animationSet = new AnimationSet(true);
	                RotateAnimation rotateAnimation = new RotateAnimation(0, 360,
	                        Animation.RELATIVE_TO_SELF, 0.5f,
	                        Animation.RELATIVE_TO_SELF, 0.5f);
	
	                rotateAnimation.setDuration(1000);
	                animationSet.addAnimation(rotateAnimation);
	                image.startAnimation(animationSet);
	
	            }
	        });
	
	        scale_btn.setOnClickListener(new View.OnClickListener() {
	            @Override
	            public void onClick(View v) {
	
	                AnimationSet animationSet = new AnimationSet(true);
	                ScaleAnimation scaleAnimation = new ScaleAnimation(
	                        0, 0.1f, 0, 0.1f,
	                        Animation.RELATIVE_TO_SELF, 0.5f,
	                        Animation.RELATIVE_TO_SELF, 0.5f
	                );
	                scaleAnimation.setDuration(1000);
	                animationSet.addAnimation(scaleAnimation);
	                image.startAnimation(animationSet);
	
	            }
	        });
	
	        alpha_btn.setOnClickListener(new View.OnClickListener() {
	            @Override
	            public void onClick(View v) {
	
	                AnimationSet animationSet = new AnimationSet(true);
	                AlphaAnimation alphaAnimation = new AlphaAnimation(1, 0);
	                alphaAnimation.setDuration(1000);
	                animationSet.addAnimation(alphaAnimation);
	                image.startAnimation(alphaAnimation);
	
	            }
	        });
	
	        translate_btn.setOnClickListener(new View.OnClickListener() {
	            @Override
	            public void onClick(View v) {
	                AnimationSet animationSet = new AnimationSet(true);
	                TranslateAnimation translateAnimation = new TranslateAnimation(
	                        Animation.RELATIVE_TO_SELF, 0f,
	                        Animation.RELATIVE_TO_SELF, 0.5f,
	                        Animation.RELATIVE_TO_SELF, 0f,
	                        Animation.RELATIVE_TO_SELF, 0.5f
	                );
	                translateAnimation.setDuration(1000);
	                animationSet.addAnimation(translateAnimation);
	                image.startAnimation(translateAnimation);
	            }
	        });
	    }
	
	
	    @Override
	    public boolean onCreateOptionsMenu(Menu menu) {
	        // Inflate the menu; this adds items to the action bar if it is present.
	        getMenuInflater().inflate(R.menu.menu_main, menu);
	        return true;
	    }
	
	    @Override
	    public boolean onOptionsItemSelected(MenuItem item) {
	        // Handle action bar item clicks here. The action bar will
	        // automatically handle clicks on the Home/Up button, so long
	        // as you specify a parent activity in AndroidManifest.xml.
	        int id = item.getItemId();
	
	        //noinspection SimplifiableIfStatement
	        if (id == R.id.action_settings) {
	            return true;
	        }
	
	        return super.onOptionsItemSelected(item);
	    }
	}
