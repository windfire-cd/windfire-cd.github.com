---
layout: post
title: "Spring入门到精通笔记"
date: 2012-08-07 11:00
comments: true
categories: 
---

##概述

模块划分

- 核心是控制反转，通过配置文件完成业务对象间的依赖注入。
- 提供事物处理功能
- 简单有效的JDBC应用
- 强大灵活的Web框架

特点

- 良好的分层
- 以IOC为核心，促使开发人员面向接口编程
- 良好架构设计
- 代替EJB
- MVC代替MVC2
- 和Struts, Hibernate结合

##安装

**安装Eclipse及插件**

下载eclipse安装

Help->Install New Software
添加插件库
http://download.eclipse.org/webtools/repository/indigo/
然后进行安装

**ubuntu安装tomcat**

**设置eclipse及tomcat**

[Setting up a Tomcat server in Eclipse](https://github.com/openplans/OpenTripPlanner/wiki/SettingUpATomcatServerInEclipse)

```bash
sudo apt-get install tomcat7
cd /usr/share/tomcat7
sudo ln -s /var/lib/tomcat7/conf conf
sudo ln -s /etc/tomcat7/policy.d/03catalina.policy conf/catalina.policy
sudo ln -s /var/log/tomcat7 log
sudo chmod -R 777 /usr/share/tomcat7/conf
```

[Installation](https://help.ubuntu.com/12.04/serverguide/tomcat.html)

[Config with Eclipse](https://github.com/openplans/OpenTripPlanner/wiki/SettingUpATomcatServerInEclipse)

##HelloWorld

###基础例子

**创建工程**

创建一个java工程

**添加依赖库**

需要在build path中添加

- commons-logging-1.1.1.jar
- log4j-1.2.16.jar
- slf4j-api-1.6.1.jar
- slf4j-log4j12-1.6.4.jar
- spring-asm-3.2.0.M1.jar
- spring-beans-3.2.0.M1.jar
- spring-context-3.2.0.M1.jar
- spring-core-3.2.0.M1.jar
- spring-expression-3.2.0.M1.jar

**添加代码**

``` java HelloWorld.java
package Hello;

public class HelloWorld {

	public String msg = null;

	public void setMsg(String s){
		this.msg = s;
	}

	public String getMsg(){
		return msg;
	}
}
```

``` java TestHello.java
package Hello;


import org.springframework.context.ApplicationContext;
import org.springframework.context.support.FileSystemXmlApplicationContext;
import Hello.HelloWorld;

public class TestHello {
	public static void main(String[] args) throws Exception{
		ApplicationContext helloContext = new FileSystemXmlApplicationContext(
				"config.xml");
		HelloWorld hw = (HelloWorld)helloContext.getBean("HelloWorld");
		System.out.println(hw.getMsg());
	}

}

```

**配置log4j以及config.xml**

```xml log4j.properties
log4j.rootLogger=DEBUG, A1
log4j.appender.A1=org.apache.log4j.ConsoleAppender
log4j.appender.A1.layout=org.apache.log4j.PatternLayout
log4j.appender.A1.layout.ConversionPattern=%-4r %-5p [%t] %37c %3x - %m%n
```

```xml config.xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE beans PUBLIC "-//SPRING//DTD BEAN//EN"
"http://www.springframework.org/dtd/spring-beans.dtd">
<beans>
	<bean id="HelloWorld" class="Hello.HelloWorld">
		<property name="msg">
			<value>HelloWorld</value>
		</property>
	</bean>
</beans>
```
其中

- *id* 		唯一标识bean
- *class*	标识bean来源
- *msg*		和<value>一起定义注入字符串，注意setAttr/getAttr的名称定义，参见 [Java Bean 属性命名规范问题分析](http://blog.csdn.net/yunye114105/article/details/7364264)

###改写例子

定义接口
```java HelloInter.java
package Hello;

public interface HelloInter {
		public String doSalutation();
}
```

实现EnHello和CnHello

```java ChHello.java
package Hello;

public class ChHello implements HelloInter{
	public String msg = null;
	public void setMsg(String msg){
		this.msg = msg;
	}
	public String getMsg(){
		return msg;
	}

	public String doSalutation(){
		return "你好" + this.msg;
	}
}

```

```java EnHello.java
package Hello;

public class EnHello implements HelloInter{
	private String msg = null;
	public void setMsg(String s){
		this.msg = s;
	}
	public String getMsg(){
		return this.msg;
	}
	public String doSalutation(){
		return "Welcome" + this.msg;
	}
}
```

更新TestHello

```java TestHello.java
package Hello;


import org.springframework.context.ApplicationContext;

public class TestHello {
	public static void main(String[] args) throws Exception{
		ApplicationContext helloContext = new FileSystemXmlApplicationContext(
				"config.xml");
		HelloWorld hw = (HelloWorld)helloContext.getBean("HelloWorld");
		System.out.println(hw.getMsg());

		System.out.println("\n============\n");

		HelloInter ch = (HelloInter)helloContext.getBean("CnHello");
		System.out.println(ch.doSalutation());

		System.out.println("\n============\n");

		HelloInter eh = (HelloInter)helloContext.getBean("EnHello");
		System.out.println(eh.doSalutation());
	}

}
```

修改config.xml

```xml config.xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE beans PUBLIC "-//SPRING//DTD BEAN//EN"
"http://www.springframework.org/dtd/spring-beans.dtd">
<beans>
<bean id="HelloWorld" class="Hello.HelloWorld">
<property name="msg">
<value>HelloWorld</value>
</property>
</bean>
<bean id="CnHello" class="Hello.ChHello">
<property name="msg" value='嗨'></property></bean>
<bean id="EnHello" class="Hello.EnHello">
<property name="msg" value="Hi"></property></bean>
</beans>
```

