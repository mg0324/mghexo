---
title: xfire-1.2.6.jar和spring版本包冲突
date: 2017-02-17 13:56:27
categories: 异常处理
tags:
- 异常
- 包冲突
---

## 问题场景描述
>最近遇到一个异常，狠狠地坑了我连续10小时。
该问题描述：
	（环境）
	1.maven的jeesite（spring+mybatis+shiro）环境下。
	2.jetty:run能够正常启动。
	3.放到tomcat上，运行报错。eclipse中的server的tomcat也一样报错。
	4.之前该项目部分多是内容发布部分，tomcat能正常启动。
		后续的业务有对接医院的业务，而且还使用了阿里大于的短信服务。
	（报错）

	如下便贴上错误日志：
	二月 17, 2017 11:41:11 上午 org.apache.catalina.core.ApplicationContext log
	信息: Loading Spring root WebApplicationContext
	二月 17, 2017 11:41:11 上午 org.apache.catalina.core.StandardContext listenerStart
	严重: Exception sending context initialized event to listener instance of class com.thinkgem.jeesite.modules.sys.listener.WebContextListener
	java.lang.NullPointerException
		at org.springframework.core.io.support.PathMatchingResourcePatternResolver.doFindPathMatchingJarResources(PathMatchingResourcePatternResolver.java:331)
		at org.springframework.core.io.support.PathMatchingResourcePatternResolver.findPathMatchingResources(PathMatchingResourcePatternResolver.java:262)
		at org.springframework.core.io.support.PathMatchingResourcePatternResolver.getResources(PathMatchingResourcePatternResolver.java:202)
		at org.springframework.context.support.AbstractApplicationContext.getResources(AbstractApplicationContext.java:681)
		at org.springframework.beans.factory.support.AbstractBeanDefinitionReader.loadBeanDefinitions(AbstractBeanDefinitionReader.java:141)
		at org.springframework.web.context.support.XmlWebApplicationContext.loadBeanDefinitions(XmlWebApplicationContext.java:126)
		at org.springframework.web.context.support.XmlWebApplicationContext.loadBeanDefinitions(XmlWebApplicationContext.java:94)
		at org.springframework.context.support.AbstractRefreshableApplicationContext.refreshBeanFactory(AbstractRefreshableApplicationContext.java:89)
		at org.springframework.context.support.AbstractApplicationContext.refresh(AbstractApplicationContext.java:269)
		at org.springframework.web.context.support.AbstractRefreshableWebApplicationContext.refresh(AbstractRefreshableWebApplicationContext.java:134)
		at org.springframework.web.context.ContextLoader.createWebApplicationContext(ContextLoader.java:246)
		at org.springframework.web.context.ContextLoader.initWebApplicationContext(ContextLoader.java:184)
		at org.springframework.web.context.ContextLoaderListener.contextInitialized(ContextLoaderListener.java:49)
		at org.apache.catalina.core.StandardContext.listenerStart(StandardContext.java:4729)
		at org.apache.catalina.core.StandardContext.startInternal(StandardContext.java:5167)
		at org.apache.catalina.util.LifecycleBase.start(LifecycleBase.java:150)
		at org.apache.catalina.core.ContainerBase$StartChild.call(ContainerBase.java:1408)
		at org.apache.catalina.core.ContainerBase$StartChild.call(ContainerBase.java:1398)
		at java.util.concurrent.FutureTask.run(FutureTask.java:266)
		at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1142)
		at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:617)
		at java.lang.Thread.run(Thread.java:745)

## 卡住的原因
>一般的，maven项目都会集成jetty,tomcat等容器插件来运行项目，只有等到部署到服务器上才会使用独立的tomcat来运行。
但是，在开发环境下jetty,tomcat等容器都是能够正常运行的，刚好昨天给客户部署发现突然报上面的错误，整个人都不好了。

为什么卡主连续10小时，是因为不好找原因。只有独立tomcat下是跑不起来的。
jetty能跑，肯定代码是没有问题的。（不过没找到原因之前，我是没有这样排除的。）
当时是到处找原因，不过也是排除法来的。至少排除了不是代码的问题引起的，只知道是环境问题。
至于是操作系统，Java环境，tomcat环境，还是项目jar包环境，不清楚。就一直尝试，一直启动。
活活卡我10小时。

## 解决的大体思路
>看异常，是报null指针，而且是在spring的底层加载包中。这说明spring加载失败了，于是我就修改web.xml中的配置。
1.先只是加载一个spring的配置文件，spring-content.xml。（修改web.xml的加载配置文件配置）
接着报了一个这个错，我就去百度。

	信息: Loading Spring root WebApplicationContext
	2017-02-17 17:10:29,843 ERROR [org.springframework.web.context.ContextLoader] - Context initialization failed
	org.springframework.beans.factory.BeanDefinitionStoreException: Line 15 in XML document from URL [file:/D:/apache-tomcat-8.0.26-eclipse/wtpwebapps/zhongliu/WEB-INF/classes/spring-context.xml] is invalid; nested exception is org.xml.sax.SAXParseException: Document root element "beans", must match DOCTYPE root "null".
	org.xml.sax.SAXParseException; lineNumber: 15; columnNumber: 27; Document root element "beans", must match DOCTYPE root "null".
		at org.apache.xerces.util.ErrorHandlerWrapper.createSAXParseException(Unknown Source)

>然后就搜到了这篇blog,http://www.iteye.com/problems/51892 终于看到了问题的所在。

<img src="https://mg0324.github.io/images/xfire-spring-ct.png" style="width:450px;"/>

>终于意识到我也是在调一个webservice的时候加入了xfire的包，于是就去找解决xfire包的冲突问题。


	<dependency>
		<groupId>org.codehaus.xfire</groupId>
		<artifactId>xfire-all</artifactId>
		<version>1.2.6<ersion>
		<exclusions>
			<exclusion>
				<groupId>org.springframework</groupId>
				<artifactId>spring</artifactId>
				<version>1.2.6<ersion>
			</exclusion>
		</exclusions>
	</dependency>

>最后问题解决，当时我很高兴，从未如此的放松。
写此博客，谨记。

