+++
date = "2015-05-02T19:55:15+08:00"
menu = "main"
tags = ["java"]
title = "Java 面试知识点"

+++

##### 基础知识  

1. 8个基本类型：  boolean, byte, short, int, float, double, long

2. 线性表种类， 区别， 优缺点： 数组， 链表； 定长， 不定长； 数组便于查找， 链表便于插入

3. jdk, jre, jvm 有什么关系： jdk里面包含jre, jre里面包含jvm

4. 抽象类和接口有什么区别： 抽象类中可以有非抽象方法和成员变量， 接口中全部是抽象方法， 变量默认是 `public static final` ; 抽象类中更加强调属性， 而接口中更加强调方法

5. ArrayList 和 LinkerList 有什么区别： ArrayList 使用数组来实现， LinkedList 使用链表来实现

6. collections 和 collection 有什么区别： collection 是一个接口， 子接口有 Queue, Set, List 分别是队列， 集合和表； collections 是 collection 的一个工具类， 获得collection 的子类的一些信息

7. 堆和栈什么区别： 堆一般是使用树实。 java 把内存分为两种： 堆内存， 栈内存  
   栈内存： 可以数据共享， 函数中定义的一些基本类型的变量和对象的引用通过栈内存进行分配  
   堆内存： 不可以数据共享， new 关键字创建的对象和数组存放在堆内存中， java 中自动垃圾回收机制回收掉  

8. 当 new 一个对象时，域， 静态域， 构造方法的调用顺序 

9. Object 的方法都有哪些? 

	clone() 创建并返回此对象的一个副本
	
	equals() 指示某个其他对象是否与此对象 “相等”

	hashCode() 返回该对象的哈希码值，这个值用来指示对象在内存中的唯一位置

	notify() 唤醒在此对象监视器上等待的单个线程

	notifyAll() 唤醒再次对象监视器上等待的所有线程

	toString() 返回该对象的字符串表示

	wait() 一系列重载函数，导致当前的线程等待。

10. Exception 和 Error 的区别： 都是继承了 Threadable 类，Exception 可以抛出，Error 不能抛出

11. JVM的工作原理：

	操作系统装入jvm是通过jdk中的java.exe来完成的，通常由下面四个步骤：
	1. 创建 jvm 装载环境和配置
	2. 装载 jvm.dll
	3. 初始化 jvm.dll 并挂接到 JNIENV（JNI调用接口）实例
	4. 调用 JNIENV 实例装载并处理 class 类

12. ClassLoader 的工作原理： 当运行一个程序的时候，JVM 启动，运行 bootstrap classloader， 该 classloader 加载 java 核心 API（ExtClassLoader和AppClassLoader也在此时加载），然后调用 ExtClassLoader 加载扩展 API，最后 AppClassLoader 加载 CLASSPATH 目录下定义的 class。这就是一个程序最基本的加载流程。

13. final，finally，finalize 的区别：

	final 是变量修饰符，被它修饰的变量只能赋值一次。

	finally 是与 try, catch 一起使用的用于异常抛出的关键字，只要执行了 try 或者 catch 中的代码，finally 一定会执行

		
		try{
			return;
		}
		catch(Exception e)
		{}
		finally{
			System.out.println("一定能执行")；
		}
	
	
	`finalize()` 方法是 GC 运行机制的一部分，finalize() 方法的是在 GC 清理它所从属的对象时被调用的，如果执行它的过程中抛出了无法捕获的异常，GC 将终止对该对象的清理。知道下一次 GC 开始清理这个对象时，它的 finalize 才会被再次调用。所以一般可以在 finalize 方法里 __ 释放系统资源或者做其他的清理工作，例如关闭输入输出流 __ 。 	

### 线程安全

1. String, StringBuilder, StringBuffer 有什么区别： String 是字符串常量，使用 final 定义的， StringBuilder 和 StringBuffer 是字符串类， StringBuffer 是线程安全的

2. 集合类中有哪些是线程安全的：   
   Vector 线程安全， ArrayList, LinkedList 线程不安全  
   HashTable 线程安全， HashMap线程不安全  
   Stack 线程安全  
   剩下的都不安全

3. synchronized 关键字如何使用  
   声明方法： 只有拥有该方法所在的对象才能调用

	
		public synchronized void sale(){
		  ....
		}
	 
   
	声明块： 可以指定调用的对象
	
		
		for (int i=0; i<10; i++){
		 synchronized(this){
		 	...
		 }
		}
		


### 垃圾回收

1. 垃圾回收机制简介： GC 是垃圾回收的意思（Gabage Collection），内存处理是编程人员容易出现问题的地方，忘记或者错误的内存回收会导致后内存的不稳定甚至崩溃，Java 提供的 GC 功能可以自动监测对象是否超过作用域从而达到自动回收内存的目的，Java 语言没有提供释放已分配内存的显示操作方法。

2. 垃圾回收算法

### Java高级特性

1. 序列化： 对象持久化。能够将对象转化成字节流，方便对象的存储或者网络传输

	Serializable： 继承这个类的类A可以通过 ObjectOuputStream 来持久化对象， 通过 ObjectInputStream 来读出这个对象

	Externalizable: 实现这个接口，重写 `wirteExternal()` 和 `readExternal()` 来进行序列化和反序列化的可控（说白了就是可以省了 ObjectOutputStream 这些……）

2. 反射： 能够通过一个类的对象得到这个类的全部信息

### 自由发挥

1. `String str = new String("abc")` 总共创建了几个对象？  3个

2. 如何使用数组实现链表？  1. 数组里面存对象。 2. 数组的下标作为链表的索引

3. 如何判断一个链表有环？  1.定义2个指针， 一个每次向后走1 步， 一个每次向后走2 步， 链表有环， 两个指针在某一时刻一定会相遇

4. 写一个单例模式

5. 写一个堆排序