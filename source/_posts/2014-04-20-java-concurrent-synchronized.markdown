---
layout: post
title: "java并发--Synchronized关键字"
date: 2014-04-20 11:13:01 +0800
comments: true
categories: 
---

待解达的疑惑：

* synchronized可以使用的应用场景及其用途
* synchronized的内部实现
* synchronized存在的问题

#### 参考文献 ####

* [Java Programming Keyword Synchronized](http://en.wikibooks.org/wiki/Java_Programming/Keywords/synchronized)
* [5-things-you-didnt-know-about-synchronization-in-java-and-scala](http://www.takipiblog.com/2013/08/15/5-things-you-didnt-know-about-synchronization-in-java-and-scala/)
* [聊聊并发（二）——Java SE1.6中的Synchronized](http://www.infoq.com/cn/articles/java-se-16-synchronized)


#### 应用场景 ####
* 同步代码块
* 同步方法

同步代码块

```
synchronized(<object_reference>) {
   // Thread.currentThread() has a lock on object_reference. All other threads trying to access it will
   // be blocked until the current thread releases the lock.
}
```

同步方法

```
public synchronized void method() {
   // Thread.currentThread() has a lock on this object, i.e. a synchronized method is the same as
   // calling { synchronized(this) {…} }.
}
```

两种使用方法都是保证一个线程能执行相关代码

<strong>注意</strong>

* synchronized总是与对象关联
* 如果方法是静态方法，则关联对象是class对象
* 如果方法是非静态方法，则关联对象是当前实例
* <strong>允许使用synchronized修饰abstract方法是没有意义的，因为synchronized是属于实现的一部分</strong>

参考单例的实现方式

```
/**
 * The singleton class that can be instantiated only once with lazy instantiation
 */
public class Singleton {
    /** Static class instance */
    private volatile static Singleton instance = null;
 
    /**
     * Standard private constructor
     */
    private Singleton() {
        // Some initialisation
    }
 
    /**
     * Getter of the singleton instance
     * @return The only instance
     */
    public static Singleton getInstance() {
        if (instance == null) {
            // If the instance does not exist, go in time-consuming
            // section:
            synchronized (Singleton.class) {
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
 
        return instance;
    }
}
```

####synchronized的内部实现####
* 两条字节码：MonitorEnter和MonitorExit
* Java 1.6以后synchronized性能已经得到一定的[优化](http://www.infoq.com/cn/articles/java-se-16-synchronized)

####synchronized存在的问题####
* 扩展性
	* synchronized 方法或语句的使用提供了对与每个对象相关的隐式监视器锁的访问，但却强制所有锁获取和释放均要出现在一个块结构中












