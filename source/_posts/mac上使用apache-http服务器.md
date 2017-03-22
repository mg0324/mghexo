---
title: 'mac上使用apache,http服务器'
date: 2016-11-04 17:04:02
categories: http服务器
tags:
- apache
- httpd
---

## 安装Apache服务器

	对于mac osx的系统，一般会预装好apache的httpd服务器的。

	所以只需要使用启动和停止命令来操作。

	如果没有安装httpd服务器，可以使用
	yum install httpd来安装（在centos下），
	apt-get install httpd（ubuntu下），
	brew install httpd（mac osx下先安装brew的包管理工具）

	sudo apachectl start - 启动httpd服务器
	sudo apachectl stop - 停止httpd服务器

## 修改配置为mgwiki

	apache是一个基金会的名字，而这款服务器叫做httpd的服务器，用来提供http的服务。还可以扩展php的一些插件，来支持php等环境。
	不过这里，我就是把她当做一个http文件服务器，用来搭一个轻文档服务amwiki。

	如果不配置虚拟主机，那么直接访问 http://localhost:80的话，是可以看到it works！的字样的，这个说明httpd启动成功。

	apache httpd的配置在/etc/apache2这个目录下，如下图：

![](https://mg0324.github.io/images/apache2-folder.png)

	只需要修改httpd.conf中的文档目录，就可以全局更换文档路径了。

![](https://mg0324.github.io/images/httpd-conf.png)

	然后重启，访问http://localhost就可以看到文档界面了。因为该产品一切都是在浏览器端完成，也就是前端，
	所以本地选用apache服务器，其实也可以选tomcat，但是感觉tomcat多是服务java的，而且要去目录里启动，
	少许麻烦，故选择了apache httpd。

	/Users/meigang/git/mgwiki 路径下是来自github上的一个开源产品amwiki。

![](https://mg0324.github.io/images/apache-mgwiki.png)

## 鸣谢
感谢：<a href="https://github.com/TevinLi/amWiki">https://github.com/TevinLi/amWiki</a>

部署自己的轻文档：<a href="https://mg0324.github.io/mgwiki/">https://mg0324.github.io/mgwiki/</a>

迁移到hexo博客，地址：<a href="https://mg0324.github.io/">https://mg0324.github.io/</a>
