+++
date = "2017-08-12T23:01:45+08:00"
menu = "main"
tags = ["life", "code"]
title = "Java 中 Thread 与 Runnable 分析"

+++



众所周知，Thread 是类，而 Runnable 是接口，Thread 实现了 Runnable 接口。Thread 有多种构造方法，其中最典型的的 ```Thread()``` 和 ```Thread(Runnable target)``` 源码如下：

```java
public Thread() {
	init(null, null, "Thread-" + nextThreadNum(), 0);
}
public Thread(Runnable target) {
	init(null, target, "Thread-" + nextThreadNum(), 0);
}
```

区别在于调用 ```init()``` 方法时参数 ```target``` 是否为 ```null``` ，在 ```init()``` 对于 ```target``` 的处理如下：

```java
private void init(ThreadGroup g, Runnable target, String name,
                      long stackSize, AccessControlContext acc) {
  ...
  this.group = g;
  this.daemon = parent.isDaemon();
  this.priority = parent.getPriority();
  if (security == null || isCCLOverridden(parent.getClass()))
      this.contextClassLoader = parent.getContextClassLoader();
  else
      this.contextClassLoader = parent.contextClassLoader;
  this.inheritedAccessControlContext =
  acc != null ? acc : AccessController.getContext();
  this.target = target;
  ....
}
```

只是将 ```target``` 赋值给了 ```Thread``` 的一个成员变量 ```this.target``` ，直到 ```Thread``` 调用 ```start()``` 函数也未曾对 ```target``` 进行单独的操作，我们暂且认为对 ```target``` 的操作是在 ```JNI``` 中。

那么我们在使用 ```Thread``` 到底用哪个构造方法更好呢，下面通过一个具体的例子解释一下：假设有20张票，我想通过3个线程购票，那么如何操作呢。

```java
public class ThreadRunnableTest {
	public static void main(String[] args) {
		Runnable runnable = new Runnable() {
			int ticket = 20;
			@Override
			public void run() {
				// TODO Auto-generated method stub
				for (int i = 0; i < 20; i++) {
                  if (ticket > 0) {
                    try {
                      Thread.sleep(100);
                    } catch (InterruptedException e) {
                      // TODO: handle exception
                      e.printStackTrace();
                    }
                    System.out.println(Thread.currentThread().getName() + " sell ticket " + ticket--);
                  }
				}
			}
		};
		
		Thread t1 = new Thread(runnable, "t1");
		Thread t2 = new Thread(runnable, "t2");
		Thread t3 = new Thread(runnable, "t3");
		
		t1.start();
		t2.start();
		t3.start();
	}
}
```

但是上述的代码输出却出现了线程不同步的问题，其中 ticket 17 出现了3次。这种现象的原因就是多个 ```Thread``` 在调用同一个 ```runnable.run()``` 的时候没有进行同步。

>t3 sell ticket 20
>
>t1 sell ticket 18
>
>t2 sell ticket 19
>
>t2 sell ticket 17
>
>t1 sell ticket 17
>
>t3 sell ticket 16
>
>t2 sell ticket 15
>
>t1 sell ticket 14
>
>t3 sell ticket 13
>
>t2 sell ticket 12
>
>t1 sell ticket 11
>
>t3 sell ticket 10
>
>t2 sell ticket 9
>
>t1 sell ticket 8
>
>t3 sell ticket 7
>
>t2 sell ticket 6
>
>t1 sell ticket 5
>
>t3 sell ticket 4
>
>t2 sell ticket 3
>
>t3 sell ticket 2
>
>t1 sell ticket 1
>
>t2 sell ticket 0

既然系统没有对 ```Runnable``` 进行同步操作，我们可以自己去实现

```java
public class ThreadRunnableTest {
	public static void main(String[] args) {
		Runnable runnable = new Runnable() {
			int ticket = 20;
			@Override
			public void run() {
				// TODO Auto-generated method stub
				for (int i = 0; i < 20; i++) {
					synchronized (this) {
						if (ticket > 0) {
							System.out.println(Thread.currentThread().getName() + " sell ticket " + ticket--);
						}
					}
                  try {
                    Thread.sleep(100);
                  } catch (InterruptedException e) {
                    // TODO: handle exception
                    e.printStackTrace();
                  }
				}
			}
		};
		
		Thread t1 = new Thread(runnable, "t1");
		Thread t2 = new Thread(runnable, "t2");
		Thread t3 = new Thread(runnable, "t3");
		
		t1.start();
		t2.start();
		t3.start();
	}
}

```

这样得到的结果是正确的

>t1 sell ticket 20
>
>t3 sell ticket 19
>
>t2 sell ticket 18
>
>t2 sell ticket 17
>
>t1 sell ticket 16
>
>t3 sell ticket 15
>
>t2 sell ticket 14
>
>t3 sell ticket 13
>
>t1 sell ticket 12
>
>t2 sell ticket 11
>
>t1 sell ticket 10
>
>t3 sell ticket 9
>
>t3 sell ticket 8
>
>t1 sell ticket 7
>
>t2 sell ticket 6
>
>t3 sell ticket 5
>
>t2 sell ticket 4
>
>t1 sell ticket 3
>
>t2 sell ticket 2
>
>t3 sell ticket 1

有趣的是如果我们把同步的代码块放在 ```if(ticket > 0)``` 语句的下面，就会发现偶尔结果多打印了一行

>t1 sell ticket 0

这是因为我们对变量 ```ticket``` 进行操作时分为“读”，“更改”，“写入” 。假设 ticket 为1时，当 t3 拿到同步锁变量并对 ticket 进行减1操作后，还没有把0写入内存，此时 cpu 调度切换到执行线程 t1，t1 读取到的 ticket 值仍为 1 ，通过了条件判断，但是需要等待 t3 同步块执行完释放同步锁。t1 拿到同步锁之后再次对 ticket 进行读取此时 ticket 已经为 0 。所以才有了上述多余的一行。

综上所述，```Thread``` 与 ```Runnable``` 之间除了是类与接口，接口与接口的实现之外，并没有其他特别主要注意的地方，当有类似上述情景的需求时，使用 一个 ```Runnable``` + 多个 ```Thread``` 是一个不错的解决方案，但是需要注意同步问题。这里就会有人问了，为啥不直接在 ```runnable``` 的 ```run()``` 方法前加一个 ```synchronized``` 关键字呢。如果加了这个关键字之后呢，只有拿到 ```runnable``` 这个对象锁的线程才能执行 ```run()``` 方法，在循环中直到 ticket 卖完。

参考

[java线程系列---Runnable和Thread的区别](http://blog.csdn.net/wwww1988600/article/details/7309070) 这篇文章有不少错误之处，注意看评论。

[java线程系列---Runnable和Thread的区别、线程同步](http://blog.csdn.net/guoxiaolongonly/article/details/50717574)