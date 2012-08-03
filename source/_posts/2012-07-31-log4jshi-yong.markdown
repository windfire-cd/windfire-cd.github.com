---
layout: post
title: "log4j使用"
date: 2012-07-31 14:42
comments: true
categories: 
---

介绍log4j使用，主要转载[Log4j使用指南](http://www.cnblogs.com/licheng/archive/2008/08/23/1274566.html)

##概述

本文档是针对Log4j日志工具的使用指南。包括：日志介绍、日志工具介绍、Log4j基本使用、Log4j的高级使用、Spring与log4j的集成等。并进行了举例说明。

##介绍

日志工具支持级别配置、输出格式配置、输出源配置等功能。工具包括：logger4j、Commons Logging、Simple Log 、MonoLog

Log4j是Apache的一个开放源代码项目，通过使用Log4j，我们可以控制日志信息输送的；我们也可以控制每一条日志的输出格式；通过定义每一条日志信息的级别，我们能够更加细致地控制日志的生成过程。最令人感兴趣的就是，这些可以通过一个配置文件来灵活地进行配置，而不需要修改应用的代码


log4j的好处在于：

1.   通过修改配置文件，就可以决定log信息的目的地——控制台、文件、GUI组件、甚至是套接口服务器、NT的事件记录器、UNIX Syslog守护进程等

2.   通过修改配置文件，可以定义每一条日志信息的级别，从而控制是否输出。在系统开发阶段可以打印详细的log信息以跟踪系统运行情况,而在系统稳定后可以关闭log输出,从而在能跟踪系统运行情况的同时,又减少了垃圾代码（System.out.println(......)等)。

3.   使用log4j，需要整个系统有一个统一的log机制，有利于系统的规划。

此外，通过Log4j其他语言接口，您可以在C、C++、.Net、PL/SQL程序中使用Log4j，其语法和用法与在Java程序中一样，使得多语言分布式系统得到一个统一一致的日志组件模块。而且，通过使用各种第三方扩展，您可以很方便地将Log4j集成到J2EE、JINI甚至是SNMP应用中。

##Log4j基本配置

Log4j由三个重要的组件构成：Loggers，Appenders和Layouts，分别表示：日志信息的优先级，日志信息的输出目的地，日志信息的输出格式。支持key=value格式设置或xml格式设置

- 日志信息的优先级从高到低有FATAL、ERROR、WARN、INFO、DEBUG，分别用来指定这条日志信息的重要程度
- 日志信息的输出目的地指定了日志将打印到控制台还是文件中
- 输出格式则控制了日志信息的显示内容

###日志信息的优先级

Log4j划分为OFF、FATAL、ERROR、WARN、INFO、DEBUG、ALL或者您定义的级别。 Log4j建议只使用四个级别，优先级从高到低分别是ERROR、WARN、INFO、DEBUG。通过在这里定义的级别，您可以控制到应用程序中相应级别的日志信息的开关。

假如在一个级别为q的Logger中发生一个级别为p的日志请求，如果p>=q,那么请求将被启用。这是Log4j的核心原则。 比如在这里定义了INFO级别，则应用程序中所有DEBUG级别的日志信息将不被打印出来。

###输出源的使用

有选择的能用或者禁用日志请求仅仅是Log4j的一部分功能。Log4j允许日志请求被输出到多个输出源。用Log4j的话说，一个输出源被称做一个Appender。 Appender包括console（控制台）, files（文件）, GUI components（图形的组件）, remote socket servers（socket 服务）, JMS（java信息服务）, NT Event Loggers（NT的事件日志）, and remote UNIX Syslog daemons（远程UNIX的后台日志服务）。它也可以做到异步记录。

一个logger可以设置超过一个的appender。 用addAppender 方法添加一个appender到一个给定的logger。对于一个给定的logger它每个生效的日志请求都被转发到该logger所有的appender上和该logger的父辈logger的appender上。

####ConsoleAppender

如果使用ConsoleAppender，那么log信息将写到Console。效果等同于直接把信息打印到System.out上了。

####FileAppender

使用FileAppender，那么log信息将写到指定的文件中。这应该是比较经常使用到的情况。相应地，在配置文件中应该指定log输出的文件名。如下配置指定了log文件名为dglog.txt

```
log4j.appender.A2.File=dglog.txt
```

注意将A2替换为具体配置中Appender的别名

####DailyRollingAppender

使用FileAppender可以将log信息输出到文件中，但是如果文件太大了读起来就不方便了。这时就可以使用DailyRollingAppender。DailyRollingAppender可以把Log信息输出到按照日期来区分的文件中。配置文件就会每天产生一个log文件，每个log文件只记录当天的log信息：

```
log4j.appender.A2=org.apache.log4j.DailyRollingFileAppender

log4j.appender.A2.file=dglog

log4j.appender.A2.DatePattern='.'yyyy-MM-dd

log4j.appender.A2.layout=org.apache.log4j.PatternLayout

log4j.appender.A2.layout.ConversionPattern= %5r %-5p %c{2} - %m%n 
```

####RollingFileAppender

文件大小到达指定尺寸的时候产生一个新的文件。

```
log4j.appender.R=org.apache.log4j.RollingFileAppender
log4j.appender.R.File= ../logs/dglog.log
# Control the maximum log file size
log4j.appender.R.MaxFileSize=100KB
# Archive log files (one backup file here)
log4j.appender.R.MaxBackupIndex=1
log4j.appender.R.layout=org.apache.log4j.PatternLayout
log4j.appender.R.layout.ConversionPattern=%p %t %c - %m%n
```

这个配置文件指定了输出源R，是一个轮转日志文件。最大的文件是100KB，当一个日志文件达到最大尺寸时，Log4J会自动把example.log重命名为dglog.log.1，然后重建一个新的dglog.log文件，依次轮转。

####WriterAppender

将日志信息以流格式发送到任意指定的地方。

###Layout的配置

Layout指定了log信息输出的样式

####布局样式

```
org.apache.log4j.HTMLLayout（以HTML表格形式布局），
org.apache.log4j.PatternLayout（可以灵活地指定布局模式），
org.apache.log4j.SimpleLayout（包含日志信息的级别和信息字符串），
org.apache.log4j.TTCCLayout（包含日志产生的时间、线程、类别等等信息）
```

####格式

```
%m 输出代码中指定的消息
%p 输出优先级，即DEBUG，INFO，WARN，ERROR，FATAL
%r 输出自应用启动到输出该log信息耗费的毫秒数
%c 输出所属的类目，通常就是所在类的全名
%t 输出产生该日志事件的线程名
%n 输出一个回车换行符，Windows平台为"rn"，Unix平台为"n"
%d 输出日志时间点的日期或时间，默认格式为ISO8601，也可以在其后指定格式，比如：%d{yyy MMM dd HH:mm:ss,SSS}，输出类似：2002年10月18日 22：10：28，921
%l 输出日志事件的发生位置，包括类目名、发生的线程，以及在代码中的行数。举例：Testlog4.main(Test Log4.java:10) 
```

####例子

例子1：显示日期和log信息

``` 
log4j.appender.A2.layout=org.apache.log4j.PatternLayout
log4j.appender.A2.layout.ConversionPattern=%d{yyyy-MM-dd HH:mm:ss,SSS} %m%n
打印的信息是：
2002-11-12 11:49:42,866 SELECT * FROM Role WHERE 1=1 order by createDate desc
```

例子2：显示日期，log发生地方和log信息

``` 
log4j.appender.A2.layout=org.apache.log4j.PatternLayout
log4j.appender.A2.layout.ConversionPattern=%d{yyyy-MM-dd HH:mm:ss,SSS} %l "#" %m%n
2002-11-12 11:51:46,313 cn.net.unet.weboa.system.dao.RoleDAO.select(RoleDAO.java:409) "#"
SELECT * FROM Role WHERE 1=1 order by createDate desc
  
例子3：显示log级别,时间,调用方法,log信息

``` 
log4j.appender.A2.layout=org.apache.log4j.PatternLayout
log4j.appender.A2.layout.ConversionPattern=[%-5p] %d{yyyy-MM-dd HH:mm:ss,SSS}
method:%l%n%m%n
log信息:
[DEBUG] 2002-11-12 12:00:57,376
method:cn.net.unet.weboa.system.dao.RoleDAO.select(RoleDAO.java:409)
SELECT * FROM Role WHERE 1=1 order by createDate desc 
```

Properties配置文件实例

``` 
log4j.rootLogger=DEBUG
#将DAO层log记录到DAOLog,allLog中
log4j.logger.DAOLog=DEBUG,A2,A4
#将逻辑层log记录到BusinessLog,allLog中
log4j.logger.Businesslog=DEBUG,A3,A4

#A1--打印到屏幕上
log4j.appender.A1=org.apache.log4j.ConsoleAppender
log4j.appender.A1.layout=org.apache.log4j.PatternLayout
log4j.appender.A1.layout.ConversionPattern=%-5p [%t] %37c %3x - %m%n

#A2--打印到文件DAOLog中--专门为DAO层服务
log4j.appender.A2=org.apache.log4j.DailyRollingFileAppender
log4j.appender.A2.file=DAOLog
log4j.appender.A2.DatePattern='.'yyyy-MM-dd
log4j.appender.A2.layout=org.apache.log4j.PatternLayout
log4j.appender.A2.layout.ConversionPattern=[%-5p] %d{yyyy-MM-dd HH:mm:ss,SSS}
method:%l%n%m%n

#A3--打印到文件BusinessLog中--专门记录逻辑处理层服务log信息
log4j.appender.A3=org.apache.log4j.DailyRollingFileAppender
log4j.appender.A3.file=BusinessLog
log4j.appender.A3.DatePattern='.'yyyy-MM-dd
log4j.appender.A3.layout=org.apache.log4j.PatternLayout
log4j.appender.A3.layout.ConversionPattern=[%-5p] %d{yyyy-MM-dd HH:mm:ss,SSS}
method:%l%n%m%n

#A4--打印到文件alllog中--记录所有log信息
log4j.appender.A4=org.apache.log4j.DailyRollingFileAppender
log4j.appender.A4.file=alllog
log4j.appender.A4.DatePattern='.'yyyy-MM-dd
log4j.appender.A4.layout=org.apache.log4j.PatternLayout
log4j.appender.A4.layout.ConversionPattern=[%-5p] %d{yyyy-MM-dd HH:mm:ss,SSS}
method:%l%n%m%n

```

##Log4j高级配置

###配置记录日志的包路径

配置Log4j.logger.com.int97=debug, 只有包为com.int97中代码的debug信息被输出到指定的输出源。

###支持日志级别继承功能

如果log4j.rootLogger=debug，其他logger默认级别为debug。可以通过配置log4j.additivity.XXX=ture/false来打开或关闭继承功能；若为 false,表示Logger 的 appender 不继承它的父Logger； 若为true，则继承，这样就兼有自身的设定和父Logger的设定。

###为不同的 Appender 设置日志输出级别

通常所有级别的输出都是放在一个文件里的，如果日志输出的级别是DEBUG级别，查找异常不是很方便。Log4j提供仅保存异常的日志功能，只需要在配置中修改Appender的Threshold 就能实现，比如下面的例子：

```
### set log levels ###
log4j.rootLogger = debug ,  stdout ,  D ,  E

### 输出到控制台 ###
log4j.appender.stdout = org.apache.log4j.ConsoleAppender
log4j.appender.stdout.Target = System.out
log4j.appender.stdout.layout = org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern =  %d{ABSOLUTE} %5p %c{ 1 }:%L - %m%n

### 输出到日志文件 ###
log4j.appender.D = org.apache.log4j.DailyRollingFileAppender
log4j.appender.D.File = logs/log.log
log4j.appender.D.Append = true
log4j.appender.D.Threshold = DEBUG ## 输出DEBUG级别以上的日志
log4j.appender.D.layout = org.apache.log4j.PatternLayout
log4j.appender.D.layout.ConversionPattern = %-d{yyyy-MM-dd HH:mm:ss}  [ %t:%r ] - [ %p ]  %m%n

### 保存异常信息到单独文件 ###
log4j.appender.D = org.apache.log4j.DailyRollingFileAppender
log4j.appender.D.File = logs/error.log ## 异常日志文件名
log4j.appender.D.Append = true
log4j.appender.D.Threshold = ERROR ## 只输出ERROR级别以上的日志!!!
log4j.appender.D.layout = org.apache.log4j.PatternLayout
log4j.appender.D.layout.ConversionPattern = %-d{yyyy-MM-dd HH:mm:ss}  [ %t:%r ] - [ %p ]  %m%n 
```

###Xml格式配置文件实例

``` html

<?xml version="1.0" encoding="GB2312"?>

<!--LOG4J CONFIGURATION - XML style -->

<!DOCTYPE log4j:configuration SYSTEM "log4j.dtd">

<log4j:configuration xmlns:log4j="http://jakarta.apache.org/log4j/">

      

<!-- STDOUT: Outputs log information to the standard output/console -->

       <appender name="STDOUT" class="org.apache.log4j.ConsoleAppender">

              <layout class="org.apache.log4j.PatternLayout">

                     <param name="ConversionPattern" value="%d %-5p - [%C{1}] %m%n"/>

              </layout>

       </appender>

      

<appender name="myApp.file.log" class="org.apache.log4j.RollingFileAppender">

              <param name="File" value="${myApp.root}/WEB-INF/logs/myApp.log"/>

              <param name="Append" value="true"/>

              <param name="MaxBackupIndex" value="3"/>

              <param name="MaxFileSize" value="2MB" />

              <layout class="org.apache.log4j.PatternLayout">

                     <param name="ConversionPattern" value="%d [%t] %p - %m%n"/>

              </layout>

       </appender>

      

       <appender name="moduleA.log"   class="org.apache.log4j.RollingFileAppender">  

        <param name="Append" value="true"   />  

        <param name="File" value="${myApp.root}/WEB-INF/logs/moduleA.log"   />  

        <param name="MaxFileSize" value="2MB"/>  

        <param name="MaxBackupIndex" value="10"   />  

        <layout class="org.apache.log4j.PatternLayout">

                     <param name="ConversionPattern" value="%d [%t] %p - %m%n"/>

              </layout>

              <filter class="org.apache.log4j.varia.StringMatchFilter">

                     <param name="StringToMatch" value=" MODULE_A _TASK_" />

                     <param name="AcceptOnMatch" value="true" />

              </filter>

    </appender>

   

    <appender name="moduleB.log"   class="org.apache.log4j.RollingFileAppender">  

        <param name="Append" value="true"   />  

        <param name="File" value="${myApp.root}/WEB-INF/logs/moduleB.log"   />  

        <param name="MaxFileSize" value="2MB"/>  

        <param name="MaxBackupIndex" value="10"   />  

        <layout class="org.apache.log4j.PatternLayout">

                     <param name="ConversionPattern" value="%d [%t] %p - %m%n"/>

              </layout>

              <filter class="org.apache.log4j.varia.StringMatchFilter">

                     <param name="StringToMatch" value="MODULE_B_TASK_" />

                     <param name="AcceptOnMatch" value="true" />

              </filter>

    </appender>

   

    <logger name="com.levinsoft.myApp.task.FileTaskThread">

           <level value="DEBUG"/>

           <appender-ref ref="moduleA.log"/>

           <appender-ref ref="moduleB.log"/>

    </logger>

   

 

    <appender name="authorization.log.debug"   class="org.apache.log4j.RollingFileAppender">  

        <param name="Append" value="true"   />  

        <param name="File" value="${myApp.root}/WEB-INF/logs/authorization_debug.log"   />  

        <param name="MaxFileSize" value="2MB"/>  

        <param name="MaxBackupIndex" value="3"   />  

        <layout class="org.apache.log4j.PatternLayout">

                     <param name="ConversionPattern" value="%d [%t] %p - %m%n"/>

              </layout>

              <filter class="org.apache.log4j.varia.LevelRangeFilter">

                     <param name="LevelMin" value="debug" />

                     <param name="LevelMax" value="debug" />

                     <param name="AcceptOnMatch" value="true" />

              </filter>

             

    </appender>

   

    <appender name="authorization.log.error"   class="org.apache.log4j.RollingFileAppender">  

        <param name="Append" value="true"   />  

        <param name="File" value="${myApp.root}/WEB-INF/logs/authorization_error.log"   />  

        <param name="MaxFileSize" value="2MB"/>  

        <param name="MaxBackupIndex" value="3"   />  

        <layout class="org.apache.log4j.PatternLayout">

                     <param name="ConversionPattern" value="%d [%t] %p - %m%n"/>

              </layout>

      

               <filter class="org.apache.log4j.varia.LevelRangeFilter">

                     <param name="LevelMin" value="error" />

                     <param name="LevelMax" value="error" />

                     <param name="AcceptOnMatch" value="true" />

              </filter>

             

    </appender>

   

    <logger name="com.levinsoft.myApp.authorization">

           <level value="DEBUG"/>

           <appender-ref ref="authorization.log.debug"/>

           <appender-ref ref="authorization.log.error"/>

    </logger>

 

       <logger name="com.levinsoft.qframe.taglib.CollectionTag">

              <level value="WARN"/>

       </logger>

      

       <appender name="moduleC_error.log"   class="org.apache.log4j.RollingFileAppender">  

        <param name="Append" value="true"   />  

        <param name="File" value="${myApp.root}/WEB-INF/logs/moduleC_error.log"   />  

        <param name="MaxFileSize" value="2MB"/>  

        <param name="MaxBackupIndex" value="10"   />  

        <layout class="org.apache.log4j.PatternLayout">

                     <param name="ConversionPattern" value="%d [%t] %p - %m%n"/>

              </layout>

               <filter class="org.apache.log4j.varia.LevelRangeFilter">

                     <param name="LevelMin" value="error" />

                     <param name="LevelMax" value="error" />

                     <param name="AcceptOnMatch" value="true" />

              </filter>

    </appender>

 

    <logger name="com.levinsoft.myApp.search.ModuleC">

              <level value="ERROR"/>

              <appender-ref ref="moduleC_error.log"/>

       </logger>

 

       <logger name="com.levinsoft">

              <level value="DEBUG"/>

              <!-- <appender-ref ref="myApp.file.log"/> -->

       </logger>

 

       <root>

              <level value="WARN"/>

              <appender-ref ref="STDOUT"/>

              <appender-ref ref="myApp.file.log"/>

              <!-- activate to log in files -->

              <!--<appender-ref ref="DAILY"/>-->

              <!--<appender-ref ref="HTML"/>-->

       </root>

</log4j:configuration>

```

##Spring对log4j的支持

Log4jConfigurer、 Log4jConfigListener、Log4jConfigServlet对dom4j进行配置或封装。org.springframework.web.util 还包括了对其他工具类的封装。

###log4j配置文件放置路径

在web.xml文件中，可以指定log4j配置文件的放置位置。缺省情况下，系统在WEB-INF/classes目录下查找。也可以在web.xml文件中指定一个位置，例如：系统将自动的进行加载。

```xml
<context-param>

              <param-name>log4jConfigLocation</param-name>

              <param-value>/WEB-INF/classes/log4j.xml</param-value>

</context-param>
```

###log4j文件中引用web.xml中的属性

在使用Spring的应用中，在web.xml可以配置spring指定的系统属性，log4j配置文件可以引用系统属性。

```

<context-param>

              <param-name>webAppRootKey</param-name>

              <param-value>usboss.root</param-value>

       </context-param>

```

```
<appender name="myApp.file.log" class="org.apache.log4j.RollingFileAppender">

              <!-- ${myApp.root}变量仅适用于Spring的配置 -->

              <param name="File" value="${myapp.root}/WEB-INF/logs/myApp.log"/>

              <param name="Append" value="true"/>

              <param name="MaxBackupIndex" value="3"/>

              <param name="MaxFileSize" value="2MB" />

              <layout class="org.apache.log4j.PatternLayout">

                     <param name="ConversionPattern" value="%d [%t] %p - %m%n"/>

              </layout>

       </appender>

```

##java使用例子

```java

import org.apache.log4j.Logger;   
 import org.apache.log4j.PropertyConfigurator;   
    
 public class Test {   
  
   public static void main(String[] args) {   
     //Get an instance of the myLogger   
     Logger myLogger = Logger.getLogger("myLogger");   
      
     //Get an instance of the childLogger   
     Logger mySonLogger = Logger.getLogger("myLogger.mySonLogger");   
     //Load the proerties using the PropertyConfigurator   
     PropertyConfigurator.configure("log4j.properties");   
  
     //Log Messages using the Parent Logger   
     myLogger.debug("Thie is a log message from the " + myLogger.getName());   
     myLogger.info("Thie is a log message from the " + myLogger.getName());   
     myLogger.warn("Thie is a log message from the " +  myLogger.getName());   
     myLogger.error("Thie is a log message from the " + myLogger.getName());   
     myLogger.fatal("Thie is a log message from the " + myLogger.getName());   
  
     mySonLogger.debug("Thie is a log message from the " + mySonLogger.getName());   
     mySonLogger.info("Thie is a log message from the " + mySonLogger.getName());   
     mySonLogger.warn("Thie is a log message from the " +  mySonLogger.getName());   
     mySonLogger.error("Thie is a log message from the " + mySonLogger.getName());   
     mySonLogger.fatal("Thie is a log message from the " + mySonLogger.getName());   
   }   
}   

```

 程序运行结果为:

        WARN - Thie is a log message from the myLogger
        ERROR - Thie is a log message from the myLogger
        FATAL - Thie is a log message from the myLogger
        WARN - Thie is a log message from the myLogger.mySonLogger
        ERROR - Thie is a log message from the myLogger.mySonLogger
        FATAL - Thie is a log message from the myLogger.mySonLogger

        另在Test.class所在的目录下看到一个log.txt文件，内容如下：

        WARN - Thie is a log message from the myLogger.mySonLogger
        ERROR - Thie is a log message from the myLogger.mySonLogger
        FATAL - Thie is a log message from the myLogger.mySonLogger

        如将配置文件log4j.properties中语句

 log4j.logger.myLogger.mySonLogger=,file

 改为

 log4j.logger.myLogger.mySonLogger=,file,console

 再次运行程序，结果如下：

        WARN - Thie is a log message from the myLogger
        ERROR - Thie is a log message from the myLogger
        FATAL - Thie is a log message from the myLogger
        WARN - Thie is a log message from the myLogger.mySonLogger
        WARN - Thie is a log message from the myLogger.mySonLogger
        ERROR - Thie is a log message from the myLogger.mySonLogger
        ERROR - Thie is a log message from the myLogger.mySonLogger
        FATAL - Thie is a log message from the myLogger.mySonLogger         
        FATAL - Thie is a log message from the myLogger.mySonLogger

        mySonLogger的日志在控制台上输出了二次，这是因为mySonLogger继承了父类console Appender，
        本身又定义了一个console Appender, 因而有二个console Appender。


