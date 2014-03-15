---
layout: post
title: "Log4j源码学习"
date: 2014-03-15 11:50:29 +0800
comments: true
categories: 
---

* 解决什么问题
	* [核心]：哪些地方需要打印日志？
	* [核心]：日志需要打印到什么地方？
	* [核心]：日志需要用什么格式打印？  
	* [优化]：不侵入业务代码的前提下，如何实现某些情况打印？某些情况不打印？
	* [优化]：不侵入业务代码的前提下，如何满足一条日志可以输出到不同渠道（文件、控制台）
	* [优化]：不侵入业务代码的情况下，增加添加输出格式，增加输出渠道等问题

---

* Log4j的解决方式
	* 不侵入业务代码的情况下，实现某些情况下打印，某些情况不打印?
		* Level机制
	* 如何满足一条日志输出到不同渠道
		* 日志和输出分离
	* 如何在不修改业务代码的情况下，增加添加输出格式，增加输出渠道等问题
		* 配置化（properties文件、xml配置）

---

### Log4j的核心组件	

<strong>[官方链接](http://logging.apache.org/log4j/1.2/manual.html)</strong>

* Logger：日志埋点（所有需要输出日志的地方）
* Appender：日志的输出渠道
* Layout：日志的格式

![UML](/images/Log4j_logger_appender_layout_uml.jpg) 

####Logger
```
// 生成Logger对象
private final Logger logger = LoggerFactory.getLogger(A.class);

// 打印日志
logger.info("info…");
logger.warn("warn…");
logger.error("error…");
```

####Appender(一般情况下不需要在程序代码中使用)

```
<appender name="CONSOLE" class="xxx.xxx.ExtendedConsoleAppender">
	<param name="Target" value="System.out" />
	<layout class="org.apache.log4j.PatternLayout">
		<param name="ConversionPattern" value="[review-web]%d %-5p [%c %L] %m%n" />
	</layout>
</appender>
```

###Log4j的Level机制

* 解决的问题：不侵入业务代码的前提下，实现某些情况下打印，某些情况不打印?
	* Logger.setLevel(Level.INFO)
	* logger.warn("warn…")
	* compare 

```
A log request of level p in a logger with (either assigned or inherited, whichever is appropriate) level q, is enabled if p >= q.
```

```
// get a logger instance named "com.foo"
Logger  logger = Logger.getLogger("com.foo");

// Now set its level. Normally you do not need to set the
// level of a logger programmatically. This is usually done
// in configuration files.
logger.setLevel(Level.INFO);

Logger barlogger = Logger.getLogger("com.foo.Bar");

// This request is enabled, because WARN >= INFO.
logger.warn("Low fuel level.");

// This request is disabled, because DEBUG < INFO.
logger.debug("Starting search for nearest gas station.");

// The logger instance barlogger, named "com.foo.Bar",
// will inherit its level from the logger named
// "com.foo" Thus, the following request is enabled
// because INFO >= INFO.
barlogger.info("Located nearest gas station.");

// This request is disabled, because DEBUG < INFO.
barlogger.debug("Exiting gas station search");) 
```

###Appender机制
* 解决的问题：不侵入业务代码的前提下，如何满足一条日志可以输出到不同渠道（文件、控制台）
	* 日志打印与输出分离
	* 一个Logger有多个Appender
	
```
// 伪代码

logger.addAppender(new ConsoleAppender());
logger.addAppender(new FileAppender());

logger.warn(msg)等价于

for (Appender appender : appenderList) {
	appender.doAppender(msg)
}
```

###配置化
* 解决的问题：不侵入业务代码的情况下，增加添加输出格式，增加输出渠道等问题
	* 配置化
	* XML配置、Properties配置

####Properties配置
```
log4j.rootLogger=debug, R

log4j.appender.R=org.apache.log4j.RollingFileAppender
log4j.appender.R.File=example.log

log4j.appender.R.layout=org.apache.log4j.PatternLayout
log4j.appender.R.layout.ConversionPattern=%p %t %c - %m%n
```

####等价的XML配置

```
<appender name="R" class="org.apache.log4j.RollingFileAppender">
   <param name="File" value="example.log" />
   <layout class="org.apache.log4j.PatternLayout">
      <param name="ConversionPattern" value="[review-web]%d %-5p [%c %L] %m%n" />
   </layout>
</appender>

<root>
   <level value="debug" />
   <appender-ref ref="R" />
</root>
```

###Bonus

####Logger的继承机制

#####Name Inheritance

* com.foo.bar ----> com.foo
* Logger logger = Logger.getLogger("com.foo.bar");
* <strong>RootLogger除外：不可以通过getLogger获取</strong>
 

#####Level Inheritance

| Logger name | Level | Inherited Level |
| ------------ | ------------- | ------------ |
| Root | LevelRoot  | LevelRoot |
| X | LevelX  | LevelX |
| X.Y | none  | LevelX |
| X.Y.Z | LevelXYZ  | LevelXYZ |

####Appender Additivity
* 默认情况下：一个Logger(C)的日志请求-->C.parent-->…-->RootLogger
* 不需要扩散：就配置Logger的Additivity为false

```
<logger name="com.ibatis" additivity="false">
   <level value="WARN" />
   <appender-ref ref="CONSOLE" />
   <appender-ref ref="ibatisAppender" />
</logger>
```

Rules of Appender Additivity

| Logger name | Added Appenders	 | Additivity Flag | Output Targets |
| ------------ | ------------- | ------------ | ------------ |
| Root | A1  | <strong>not applicable</strong> | A1 |
| X | A2  | true | A1, A2 |
| X.Y | A3 | false | A3 |
| X.Y.Z | A4 | true | A3, A4 |
