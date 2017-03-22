---
title: mysql下的内容发布中文章的上一条和下一条解决方案
date: 2016-12-30 17:37:50
categories: mysql
tags:
- mysql
- 上一条
- 下一条
---

# 问题场景

	在内容发布系统中，都会有类似下图的需求。
<img src="https://mg0324.github.io/images/lastnext.png" style="height:50px;"/>

# 遇到的问题

	其实“上一条，下一条”的需求，转化到实现其实就是数据库的sql查询上了。
	通过当前记录，查询到所在排序中的上一条记录，和下一条记录。
	在oracle中有rownum来实现，在mysql中要怎么实现呢？

	在网上查过一些资料，很多都是用 > 或者 < 来比较id（需要将排序的字段转成整数）
	来实现，虽然也可以。但是只能支持按一种标准排序，如按权重倒叙，按时间倒数；无
	法实现2者都倒叙。所以这个是缺点。

# mysql中的rownum实现
参考博客：http://www.cnblogs.com/advocate/archive/2012/03/02/2376900.html
	
	SELECT @rownum:=@rownum+1 rownum,t.id,t.title,t.weight,t.create_date From
	(SELECT @rownum:=0,a.id,a.title,a.weight,a.create_date FROM cms_article a 
	WHERE del_flag=0 ORDER BY weight desc,create_date desc limit 10) t

 截图查询如下：
<img src="https://mg0324.github.io/images/mysql-rownum.png"/>

# 基于mysql的rownum解决方案
	
	在有了rownum后，我们可以给select结果集的记录编号，可以知道编号的rownum和记录的id的
	对应关系。这样比如当前是第2条，我们传入编号2，上一条就是2-1，下一条就是2+1，很简单的
	实现了。而且，本人在参与的一个网站中，就按此解决了这种问题。特此博客分享。




