---
layout: post
title: "Log4j源码学习之--Logger.getLogger"
date: 2014-03-16 17:22:38 +0800
comments: true
categories: 
---

<!--more-->

* Logger.getLogger的执行逻辑

![时序图](/images/Log4j_logger_getLogger_timeline.jpg)

可以看出，实际的Logger创建有Hierachy完成，先贴源码

```
public Logger getLogger(String name, LoggerFactory factory) {
    
    //System.out.println("getInstance("+name+") called.");
    CategoryKey key = new CategoryKey(name);
    
    // Synchronize to prevent write conflicts. Read conflicts (in
    // getChainedLevel method) are possible only if variable
    // assignments are non-atomic.
    Logger logger;

    synchronized(ht) {
      Object o = ht.get(key);
      if(o == null) {
			logger = factory.makeNewLoggerInstance(name);
			logger.setHierarchy(this);
			ht.put(key, logger);
			updateParents(logger);
			return logger;
      } else if(o instanceof Logger) {
			return (Logger) o;
      } else if (o instanceof ProvisionNode) {
			logger = factory.makeNewLoggerInstance(name);
			logger.setHierarchy(this);
			ht.put(key, logger);
			updateChildren((ProvisionNode) o, logger);
			updateParents(logger);
			return logger;
      }
      else {
			// It should be impossible to arrive here
			return null;  // but let's keep the compiler happy.
      }
    }
}
```

####源码分析
* Hierachy内部使用Hashtable这个数据结构存储Logger
* 获取流程：
	1. 如果不存在，则创建新的Logger，并updateParents(后面描述)
	2. 如果存在，并且是Logger类型，则直接返回
	3. 如果存在，并且<strong>ProvisionNode</strong>类型，创建Logger，并updateChildren和updateParents
	
#####ProvisionNode
源码如下：

```
class ProvisionNode extends Vector {
    
  ProvisionNode(Logger logger) {
    super();
    this.addElement(logger);
  }
}
```
代码中可以看出ProvisionNode是Vector类型，用于维护它的子孙logger的指针。
	
	
####updateParents逻辑

源码：

```
private void updateParents(Logger cat) {
  String name = cat.name;
  int length = name.length();
  boolean parentFound = false;

  //System.out.println("UpdateParents called for " + name);

  // if name = "w.x.y.z", loop thourgh "w.x.y", "w.x" and "w", but not "w.x.y.z"
  for(int i = name.lastIndexOf('.', length-1); i >= 0;
                                 i = name.lastIndexOf('.', i-1))  {
    String substr = name.substring(0, i);

    //System.out.println("Updating parent : " + substr);
    CategoryKey key = new CategoryKey(substr); // simple constructor
    Object o = ht.get(key);
    // Create a provision node for a future parent.
    if(o == null) {
      //System.out.println("No parent "+substr+" found. Creating ProvisionNode.");
      ProvisionNode pn = new ProvisionNode(cat);
      ht.put(key, pn);
    } else if(o instanceof Category) {
      parentFound = true;
      cat.parent = (Category) o;
      //System.out.println("Linking " + cat.name + " -> " + ((Category) o).name);
      break; // no need to update the ancestors of the closest ancestor
    } else if(o instanceof ProvisionNode) {
      ((ProvisionNode) o).addElement(cat);
    } else {
      Exception e = new IllegalStateException("unexpected object type " +
				o.getClass() + " in ht.");
      e.printStackTrace();
    }
  }
  // If we could not find any existing parents, then link with root.
  if(!parentFound)
    cat.parent = root;
}
```

Logger.getLogger("com.foo.bar.X")的流程即：

* 依次对名字com.foo.bar,com.foo,com的logger进行下面逻辑处理
	1. 如果logger不存在，则创建ProvisionNode对象，先占据位置
	2. 如果logger存在，并且是logger类型，则直接将当前logger的parent指向该logger，并直接返回 
	3. 如果logger存在，并且是ProvisionNode，则将当前logger作为该ProvisionNode的子孙节点
* 如果没有找到parent logger， 则将当前logger的parent指向rootLogger

####updateChildren逻辑

源码：

```
private void updateChildren(ProvisionNode pn, Logger logger) {
  //System.out.println("updateChildren called for " + logger.name);
  final int last = pn.size();

  for(int i = 0; i < last; i++) {
    Logger l = (Logger) pn.elementAt(i);
    //System.out.println("Updating child " +p.name);

    // Unless this child already points to a correct (lower) parent,
    // make cat.parent point to l.parent and l.parent to cat.
    if(!l.parent.name.startsWith(logger.name)) {
      logger.parent = l.parent;
      l.parent = logger;
    }
  }
}	
```

注意：Logger.getLogger("com.foo.bar.X"),如果com.foo.bar.X对应的logger是ProvisionNode类型，则会通过updateChildren来更新子孙节点的parent logger指向

举个例子：

前置条件:

1.先调用Logger.getLogger("com.foo.bar.X");
2.然后调用Logger.getLogger("com.foo")；

执行第一步

* 名字为com.foo.bar.X的logger会被创建
* 名字为com.foo.bar、com.foo和com的logger都会被创建成ProvisionNode对象；
* com.foo.bar.X的parent会指向RootLogger

执行第二步

* 名字为com.foo的logger会被创建，假设名字为A
* com.foo.bar.X的parent会指向A
* A.parent会指向RootLogger


结论

1. logger的继承关系是通过不断调用Logger.getLogger来创建起来的，并且这样的继承关系靠Hierarchy类来维护
2. 相同名字的logger只会闯将一次（不同的ClassLoader例外）






