+++
date = "2015-10-31T19:54:46+08:00"
menu = "main"
tags = ["android"]
title = "使用timer和timertask遇到的一些问题"

+++

timer和timertask联合起来使用可以实现很好的定时任务效果，下面是一个demon。

	//true 说明这个timer以daemon(守护进程)方式运行（优先级低，程序结束timer也自动结束） 
	java.util.Timer timer = new java.util.Timer(true);

	TimerTask task = new TimerTask() {
	   public void run() {
	   //每次需要执行的代码放到这里面。   
	   }   
	};
	
	//以下是几种调度task的方法：
	
	//time为Date类型：在指定时间执行一次。
	timer.schedule(task, time);
	
	//firstTime为Date类型,period为long，表示从firstTime时刻开始，每隔period毫秒执行一次。
	timer.schedule(task, firstTime, period);   
	
	//delay 为long类型：从现在起过delay毫秒执行一次。
	timer.schedule(task, delay);
	
	//delay为long,period为long：从现在起过delay毫秒以后，每隔period毫秒执行一次。
	timer.schedule(task, delay, period);

但是使用timer会有很多隐患：

* 时间计算不准确：timer是以绝对时间计算定时任务的，因此会受到系统时间的影响。
* 单次只能执行一次任务：每次只从队列中拿出一个任务执行。
* 前面的任务出现错误的话后面的任务不会执行。

schedule()源码

	/*
	553     * Schedule a task.
	554     */
	555    private void More ...scheduleImpl(TimerTask task, long delay, long period, boolean fixed) {
	556        synchronized (impl) {
	557            if (impl.cancelled) {
	558                throw new IllegalStateException("Timer was canceled");
	559            }
	560
	561            long when = delay + System.currentTimeMillis();
	562
	563            if (when < 0) {
	564                throw new IllegalArgumentException("Illegal delay to start the TimerTask: " + when);
	565            }
	566
	567            synchronized (task.lock) {
	568                if (task.isScheduled()) {
	569                    throw new IllegalStateException("TimerTask is scheduled already");
	570                }
	571
	572                if (task.cancelled) {
	573                    throw new IllegalStateException("TimerTask is canceled");
	574                }
	575
	576                task.when = when;
	577                task.period = period;
	578                task.fixedRate = fixed;
	579            }
	580
	581            // insert the newTask into queue
	582            impl.insertTask(task);
	583        }
	584    }

#### 下面很重要

---

同一个TimerTask对象不能够被schedule两次，否则会抛出`TimerTask is scheduled already`。

当一个TimerTask被cancel的时候需要重新进行实例化才可以schedule，否则会抛出出`TimerTask is canceled`。