+++
date = "2015-10-30T16:24:45+08:00"
menu = "main"
tags = ["android"]
title = "Android属性动画cancel和end解析"

+++

最近发现看Android源码真的有很多好处，起码有些BUG不用到处Google了，这里就分析一下取消Android属性动画的一些问题。

Android的动画主要有三种：属性动画，View Animation（这个不好翻译），帧动画。

属性动画出来之后，能够让开发者很方便的完成一系列小动画的制作（比如说图片的缩小，透明，旋转，而且很大程度上甚至可以不用AnimationSet），我们使用属性动画的方式如下：

	ImageView circle = (ImageView) findViewById(R.id.circle_iv);
	ValueAnimator mRotate = ValueAnimator.ofFloat(0, -1, 0, 1, 0); // 这个就会从0,-1,0,1,0之间依次取一系列浮点数值
	mRotate.setDuration(1000); // 这个动画的单次执行周期时1s
	mRotate.setInterpolator(new LinearInterpolator()) // 这个设置插值器是线性的，表示取出的值是线性的，还有一些抛物线啦，双曲线之类的
	mRotate.setRepeatCount(ValueAnimator.INFINITE); // 这个动画轮训播放
	mRotate.setStartDelay(1000); // 这个动画延时1s
	mRotate.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
	    @Override
	    public void onAnimationUpdate(ValueAnimator animation) {
	        float value = (float) animation.getAnimatedValue(); // 这个能够获得从0~0,上面说的一系列浮点数值
	        circle.setRotation(30 * value);
	    }
	});
	mRotate.addListener(new AnimatorListenerAdapter() {
	    @Override
	    public void onAnimationCancel(Animator animation) {
	        // 当mRotate.cancel()时会回调这个函数，再回调onAnimationEnd()
	        circle.setRotation(0f);
	    }
	
	    @Override
	    public void onAnimationEnd(Animator animation) {
	        // 当mRotate.end()时会回调这个函数
	        circle.setRotation(0f);
	    }
	});
	mRotate.start();

上述的代码主要能够实现一个图片的左右一直摇摆～。

我建议使用 `AnimatorListenerAdapter` ,使用这个监听器的话不必实现四个回调,如果使用`AnimatorListener` 的话就需要实现四个完整的方法，如下

            mRotate.addListener(new ValueAnimator.AnimatorListener() {
                @Override
                public void onAnimationStart(Animator animator) {

                }

                @Override
                public void onAnimationEnd(Animator animator) {

                }

                @Override
                public void onAnimationCancel(Animator animator) {

                }

                @Override
                public void onAnimationRepeat(Animator animator) {

                }
            });
            mRotate.start();


#### 下面比较重点

--------------------

使用动画时容易造成内存泄露，尤其当我们没有及时销毁 `ValueAnimator.INFINITE` 的动画。我们需要调用一些方法将其停止，主要的是 `mRotate.cancel()` 或者 `mRotate.end()`。


我们通常会在 `AnimatorListener` 的 `onAnimationCancel()` 和 `onAnimationEnd()` 方法里做动画停止之后的操作，比如说View的恢复。

但是要注意一点**mRotate.cancel()是会先回调onAnimationCancel()然后再回调onAnimationEnd()**，这个也是我在源码中看到的。

    @Override
    1078    public void More ...end() {
    1079        AnimationHandler handler = getOrCreateAnimationHandler();
    1080        if (!handler.mAnimations.contains(this) && !handler.mPendingAnimations.contains(this)) {
    1081            // Special case if the animation has not yet started; get it ready for ending
    1082            mStartedDelay = false;
    1083            startAnimation(handler);
    1084            mStarted = true;
    1085        } else if (!mInitialized) {
    1086            initAnimation();
    1087        }
    1088        animateValue(mPlayingBackwards ? 0f : 1f);
    1089        endAnimation(handler); // 请注意这里
    1090    }

            public void More ...cancel() {
    1055        // Only cancel if the animation is actually running or has been started and is about
    1056        // to run
    1057        AnimationHandler handler = getOrCreateAnimationHandler();
    1058        if (mPlayingState != STOPPED
    1059                || handler.mPendingAnimations.contains(this)
    1060                || handler.mDelayedAnims.contains(this)) {
    1061            // Only notify listeners if the animator has actually started
    1062            if ((mStarted || mRunning) && mListeners != null) {
    1063                if (!mRunning) {
    1064                    // If it's not yet running, then start listeners weren't called. Call them now.
    1065                    notifyStartListeners();
    1066                }
    1067                ArrayList<AnimatorListener> tmpListeners =
    1068                        (ArrayList<AnimatorListener>) mListeners.clone();
    1069                for (AnimatorListener listener : tmpListeners) {
    1070                    listener.onAnimationCancel(this);
    1071                }
    1072            }
    1073            endAnimation(handler); // 还有这里
    1074        }
    1075    }

            protected void More ...endAnimation(AnimationHandler handler) {
    1157        handler.mAnimations.remove(this);
    1158        handler.mPendingAnimations.remove(this);
    1159        handler.mDelayedAnims.remove(this);
    1160        mPlayingState = STOPPED;
    1161        mPaused = false;
    1162        if ((mStarted || mRunning) && mListeners != null) {
    1163            if (!mRunning) {
    1164                // If it's not yet running, then start listeners weren't called. Call them now.
    1165                notifyStartListeners();
    1166             }
    1167            ArrayList<AnimatorListener> tmpListeners =
    1168                    (ArrayList<AnimatorListener>) mListeners.clone();
    1169            int numListeners = tmpListeners.size();
    1170            for (int i = 0; i < numListeners; ++i) {
    1171                tmpListeners.get(i).onAnimationEnd(this); // 回掉
    1172            }
    1173        }
    1174        mRunning = false;
    1175        mStarted = false;
    1176        mStartListenersCalled = false;
    1177        mPlayingBackwards = false;
    1178        mReversing = false;
    1179        mCurrentIteration = 0;
    1180        if (Trace.isTagEnabled(Trace.TRACE_TAG_VIEW)) {
    1181            Trace.asyncTraceEnd(Trace.TRACE_TAG_VIEW, getNameForTrace(),
    1182                    System.identityHashCode(this));
    1183        }
    1184    }

  
看源码的网站可以使用 [http://grepcode.com/](http://grepcode.com/)
