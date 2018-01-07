+++
date = "2015-04-17T07:20:43+08:00"
menu = "main"
tags = ["android"]
title = "Handler 机理以及使用方法"

+++

##### Handler机理以及使用方法

简单的说，handler 是一种消息机制，负责向某个消息队列中插入消息，并且对于插入的消息, handler可以使用自身的 `handleMessage()` 进行处理。handle r所在的线程必须存在一个 looper (一个 looper 类似一个消息队列循环)，这就决定了一般使用 handler 更新UI时， handler 一般是在UI线程中，一般在子线程中若是声明一个 handler 时，通常会报错，因为子线程默认是没有 looper 的，所以在子线程中声明 handler 时一般是配合 looper 使用。  

	class LooperThread extends Thread{
		public Handler mHandler;
		public void run{
			Looper.prepare(); // 创建消息循环
			mHandler = new Handler(){
				public void handleMessage(Message msg){
					// process incoming message here
				}
			}
			Looper.loop(); // 使消息循环起作用
		}
	}

这里说到的了 hanlder ，就不得不说一下 **looper** , MessageQueue 是存放Android线程中消息的数据结构，Looper 向外界提供一种访问 Message 里面对象的接口。 使用 `Looper.prepare()` 创建消息循环， `Looper.loop()` 使消息循环开始运行。因为启动的消息循环，所以写在 ` ooper.loop()` 后面的代码不会马上执行，是等待 `mHandler.getLooper().quit()` 后，looper 终止，其后的代码才能运行。  

附加使用 handler 的 `post()` 和 `handleMessage()` 方法在子线程中更新线程的方法  

1. 使用 post() 传递一个 Runnable 对象，在这个 Runnable 对象中执行更新UI或者执行回调函数  

		mHandler.post(new Runnable(){
			@Override
			public void run(){
				mTextView.setText("UI update");
			}
		}) ;

2. 在UI线程中定义一个 Handler 对象，然后再 `handlerMessage()` 中对UI进行更新或者执行回调函数  

		private Handler mHandler = new Handler(){
			@Override 
			public void handleMessage(Message msg){
				super.handleMessage();
				switch(msg.what){
					case 1:
						button1.setText("button1 update");
						break;
					case 2:
						button2.setText("button2 update");
						break;
				}
			}
		};
		
		// 发送消息
		Message message = new mHandler.obtainMessage();
		message.what = 1;
		handler.sendMessage(message);
