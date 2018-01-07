+++
date = "2015-05-22T11:44:32+08:00"
menu = "main"
tags = ["android"]
title = "Android音乐播放"

+++

android.media.MediaPlayer 是在Android实现播放音乐需要用到的类，这里先贴一张图说明MediaPlayer的生命周期

![生命周期](../../img/mediaplayer_state_diagram.gif)

主要的几个流程就是Idle->Initialized->Prepared->Started->Paused->Stoped

代码基本如下

	MediaPlayer mMediaPlayer = new MediaPlayer();
	OnComletionListener mCompletionListener = new OnCompletionListener{
		@Override
		public void onCompletion(MediaPlayer mp){
			playNext(); // 自定义的播放下一首
		}
	}
	mMediaPlayer.setDataSource(musicPath);
	mMediaPlayer.setonPreparedListener(new MediaPlayer.OnPreparedListener(){
		@Override
		public void onPrepared(MediaPlayer mp){
			mMediaPlayer.start();
		}
	})；
	mMediaPlayer.prepare();

	……

	mMediaPlayer.pause();

	……

	mMediaPlayer.stop();
	mMediaPlayer.reset();
	mMediaPlayer.release();

这里有个需要注意的地方就是mMediaPlayer.prepare()估计是在后台另开了一个线程，去准备，所以mMediaPlayer.start()可能会爆出 NullPointerException 这个异常，所以正确的做法是

	mMediaPlayer.setonPreparedListener(new MediaPlayer.OnPreparedListener(){
		@Override
		public void onPrepared(MediaPlayer mp){
			mMediaPlayer.start();
		}
	})

这样就能够等到 MediaPlayer prepare完成之后进行播放了，同样的，要想通过 MediaPlayer 获取一播放音乐的一些信息，也必须放到 onPrepared 里面

	mMediaPlayer.setonPreparedListener(new MediaPlayer.OnPreparedListener(){
		@Override
		public void onPrepared(MediaPlayer mp){
			setSeekBarTracker(mMediaPlayer.getDuration()); // 获取播放的时常，通常是以 ms 为单位的
			mMediaPlayer.start();
		}
	})

我的源码在 [MiPlayer](https://github.com/MaybeMercy/miplayer) ,大家可以去看一下