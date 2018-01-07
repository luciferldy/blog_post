+++
date = "2015-05-03T11:37:21+08:00"
menu = "main"
tags = ["java"]
title = "Java 引用"

+++

java中引用分为强引用，软引用， 弱引用， 虚引用  

1. 强引用

    平常使用最多的引用。  
    只有当分配的内存对象不再有任何引用时GC才可能开始回收期内存空间。  

    ```
    A obj = new A();
    ....
    obj = null;
    ```  

    此时obj可以被回收  

2. 软引用 softreference

    softrefrence 常常被用于高速缓存 cache， 它的特点是， 并不是你把它设成了 null， GC 就会
    回收它， 而是只有当内存不足时， 才去回收它。  

3. 弱引用 weakreference

    实际上是一种间接引用， 当我们希望能够随时取得某对象的信息， 但又不想影响此对象的垃圾收集， 那么
    你应该用 weakreference 来记住次对象。 
 
    ```
    A obj = new A();  
    WeakReference wr = new WeakReference(obj);  
    obj = null;  
    // 等待一段时间,obj对象就会被回收
    if(wr.get() == null){  
        System.out.println("obj 对象已经被清除了");  
    }
    else{  
        System.out.println("obj 对象清除， 其信息是".obj.toString());  
    }
    ```

4. 虚引用

    虚引用， 顾名思义， 就是形同虚设， 与其他几种引用都不同， 虚引用并不会决定对象的生命周期， 
    如果一个对象仅持有虚引用， 那么就和没有任何引用一样， 在任何时候都可能被垃圾回收。 虚引用主要
    用来跟踪对象被垃圾回收的活动。 虚引用必须和引用队列 (ReferenceQueue) 联合使用。