---
layout: post
title: "Logger源码学习--FileAppender"
date: 2014-03-27 16:51:26 +0800
comments: true
categories: 
---

<!--more-->

疑惑：

* 在多线程环境下，Log4j如何保证写文件的线程安全？

二话不说，先上UML图，了解一下FileAppender的继承体系

![FileAppender](/images/log4j_file-appender-uml.jpg)

### AppenderSkeleton（Abstract） ####

该类主要提供了Appender的一些常用功能，主要是下面两个功能：

* Filter的增加和删除
* 给出了Appender接口中doAppender方法的执行骨架

```
/**
* This method performs threshold checks and invokes filters before
* delegating actual logging to the subclasses specific {@link
* AppenderSkeleton#append} method.
* */
public synchronized void doAppend(LoggingEvent event) {
	if(closed) {
	  LogLog.error("Attempted to append to closed appender named ["+name+"].");
	  return;
	}

	if(!isAsSevereAsThreshold(event.getLevel())) {
	  return;
	}

	Filter f = this.headFilter;

	FILTER_LOOP:
	while(f != null) {
	  switch(f.decide(event)) {
	  case Filter.DENY: return;
	  case Filter.ACCEPT: break FILTER_LOOP;
	  case Filter.NEUTRAL: f = f.next;
	  }
	}

	this.append(event);    
}
  
```

注意：这里方法加入了<strong>synchronized</strong>关键字，表示doAppender是有线程同步的，这里可以看出这个是有一些问题的，带着这个问题，google搜索了一下[why AppenderSkeleton.doAppend is synchronized](http://mail-archives.apache.org/mod_mbox/logging-log4j-user/201003.mbox/%3C3101636D-567C-45A1-BEAA-8DC0ECD32960@apache.org%3E)

```
log4j 1.2 was designed a long time ago and relies on that big lock to provide thread safety.
 Other classes in log4j (layouts, appenders and the like) were designed assuming that they
would be externally synchronized by that lock and likely not safe if that lock is bypassed.

You could either implement the Appender interface or extend AppenderSkeleton but override
doAppend (copying and pasting the implementation but without the synchronized modifier, but
all appenders, layouts, etc used must be able to operate safely without that lock.

Addressing this issue is one of the core design goals for log4j 2.0 (http://issues.apache.org/jira/browse/LOG4J2-3)
```



流程如下

![AppenderSkeleton的执行流程](/images/AppenderSkeleton的流程处理.jpg)

```
/**
 Subclasses of <code>AppenderSkeleton</code> should implement this
 method to perform actual logging. See also {@link #doAppend
 AppenderSkeleton.doAppend} method.

 @since 0.9.0
*/
protected abstract void append(LoggingEvent event);

```

append方法由具体的Appender实现去实现

###WriterAppender###

```
/**
 This method is called by the {@link AppenderSkeleton#doAppend}
 method.

 <p>If the output stream exists and is writable then write a log
 statement to the output stream. Otherwise, write a single warning
 message to <code>System.err</code>.

 <p>The format of the output will depend on this appender's
 layout.

*/
public void append(LoggingEvent event) {
	if(!checkEntryConditions()) {
	  return;
	}
	subAppend(event);
}
```

checkEntryConditions只是做一些常规的检验

subAppend(event)的源码
```
/**
 Actual writing occurs here.

 <p>Most subclasses of <code>WriterAppender</code> will need to
 override this method.

 @since 0.9.0 */
protected void subAppend(LoggingEvent event) {
    this.qw.write(this.layout.format(event));

    if(layout.ignoresThrowable()) {
      String[] s = event.getThrowableStrRep();
      if (s != null) {
	int len = s.length;
	for(int i = 0; i < len; i++) {
	  this.qw.write(s[i]);
	  this.qw.write(Layout.LINE_SEP);
	}
      }
    }

    if(this.immediateFlush) {
      this.qw.flush();
    }
}
```

可以看出主要的写操作是代理给qw变量去做的，该变量是QuietWriter类型的，让我们来看看QuietWriter是何方神圣

```
/**
   QuietWriter does not throw exceptions when things go
   wrong. Instead, it delegates error handling to its {@link ErrorHandler}. 

   @author Ceki G&uuml;lc&uuml;

   @since 0.7.3
*/
public class QuietWriter extends FilterWriter {

  protected ErrorHandler errorHandler;

  public
  QuietWriter(Writer writer, ErrorHandler errorHandler) {
    super(writer);
    setErrorHandler(errorHandler);
  }

  public
  void write(String string) {
    try {
      out.write(string);
    } catch(IOException e) {
      errorHandler.error("Failed to write ["+string+"].", e, 
			 ErrorCode.WRITE_FAILURE);
    }
  }

  public
  void flush() {
    try {
      out.flush();
    } catch(IOException e) {
      errorHandler.error("Failed to flush writer,", e, 
			 ErrorCode.FLUSH_FAILURE);
    }	
  }


  public
  void setErrorHandler(ErrorHandler eh) {
    if(eh == null) {
      // This is a programming error on the part of the enclosing appender.
      throw new IllegalArgumentException("Attempted to set null ErrorHandler.");
    } else { 
      this.errorHandler = eh;
    }
  }
}
```

原来只是继承对FilterWriter的write方法做了容错处理

深入看了一下Writer的源码

```
/**
 * Writes a portion of a string.
 *
 * @param  str
 *         A String
 *
 * @param  off
 *         Offset from which to start writing characters
 *
 * @param  len
 *         Number of characters to write
 *
 * @throws  IndexOutOfBoundsException
 *          If <tt>off</tt> is negative, or <tt>len</tt> is negative,
 *          or <tt>off+len</tt> is negative or greater than the length
 *          of the given string
 *
 * @throws  IOException
 *          If an I/O error occurs
 */
public void write(String str, int off, int len) throws IOException {
	synchronized (lock) {
	    char cbuf[];
	    if (len <= writeBufferSize) {
		if (writeBuffer == null) {
		    writeBuffer = new char[writeBufferSize];
		}
		cbuf = writeBuffer;
	    } else {	// Don't permanently allocate very large buffers.
		cbuf = new char[len];
	    }
	    str.getChars(off, (off + len), cbuf, 0);
	    write(cbuf, 0, len);
	}
}
```

在每次写入操作的时候都有同步操作，再看一下lock对象

```
/**
 * The object used to synchronize operations on this stream.  For
 * efficiency, a character-stream object may use an object other than
 * itself to protect critical sections.  A subclass should therefore use
 * the object in this field rather than <tt>this</tt> or a synchronized
 * method.
 */
protected Object lock;

/**
 * Creates a new character-stream writer whose critical sections will
 * synchronize on the writer itself.
 */
protected Writer() {
	this.lock = this;
}
```
原来lock就是Writer对象本身

总结：

Log4j如何保证写文件多线程安全?

AppenderSkeleton本身append操作是同步过的，保证了每次append是原子操作的

Bonus：java中的Writer的写操作也是原子的（经过同步）






