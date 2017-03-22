---
title: 基于solr的全文检索解决方案
date: 2016-11-09 11:35:59
categories: 解决方案
tags:
- solr
- apache
- 全文检索
- 解决方案
---

## 思路
在一堆数据中，给其建立全文索引，然后通过查询关键字，获取匹配的数据显示到界面条目。

## 基于solr的全文检索搭建
### solr所处位置
Apache提供了全文检索引擎lucene，现已发展到6.2.X的版本。如果想自己建立搜索引擎，
可以使用lucene来加入自己的项目，作为一个全文检索模块。但是，此方法，学习成本比较大，
需要了解很多lucene的api，重点在索引建立，索引更新上。

那么如何快速拥有一个全文检索的服务呢？答案是solr,一款apache基于lucene开发的企业级
全文检索服务应用，对外提供全文检索相关接口，配置化索引建立，api提供给其他应用来访问
全文检索服务。

<img src="http://mg0324.github.io/images/solr-mind.png" style="width:300px;"/>

### solr服务器搭建
请参考:<a href="http://www.cnblogs.com/mangyang/p/5500852.html" target="_blank">http://www.cnblogs.com/mangyang/p/5500852.html</a>
    按照博客中步骤搭建好自己的tomcat-solr后，可以用solr admin的界面测试query，
    也可以程序来做插入文档，查询文档测试。


### 要解决的点
* 数据更新后，索引如何同步更新？
解决方式：
    配置定时器，周期性的更新索引。使用apache-solr-datascheduler-1.0.jar，参考 https://code.google.com/archive/p/solr-dataimport-scheduler/

<img src="http://mg0324.github.io/images/solr-asyc.png" style="width:600px;"/>

* 如何知道数据是来自哪个表的？
解决方式：
    通过solr的cores来区分。

<img src="http://mg0324.github.io/images/solr-block.png" style="width:100px;"/>

* solr 查询语法：
参考http://www.cnblogs.com/rainbowzc/p/4354224.html

## 我搭建好的solr demo
放到百度分享上：

<a href="https://pan.baidu.com/s/1kVRT0TT">hapache-tomcat-8.0.37-solr.zip</a>

这是我在mac上搭建的solr服务。
