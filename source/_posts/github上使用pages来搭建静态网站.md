---
title: github上使用pages来搭建静态网站
date: 2016-11-04 15:44:08
categories: github
tags:
- github
- github pages
---

### 1.起因 & 原理剖析

	github站点是一款流行的开源代码托管服务，使用git作为版本控制。
	而github pages是其提供的一款静态站点服务，让开发者能够将自己的静态网页在线发布出来。
	因此有很多大神把自己的个人简历，产品官网，开源框架等等静态的东西利用github pages展现在internet上。

	这里我做为一个程序猿，换电脑什么的不想丢弃一些直接写的精美文档，就捯饬了一个基于amwiki的轻文档演示站点。
	欢迎大家来踩。

	原理：之前一直想不通，为什么一些大神能够用xxx.github.io的二级域名挂上自己的站点？直到有一天，我找md文档
	解决方案的时候，找到了amwiki，是一开源产品。然后理解到其实ajax可以获取到静态资源文件，github pages本身
	就相当于提供的一个静态服务器，可以http外网访问服务器上的文件。

### 2.搭建amwiki & 自己的轻文档

* 2.1.github上建立一个仓库名为mgwiki,并git clone到自己的本地,我这里路径是~/git/mgwiki

* 2.2.去github 网站的mgwiki项目，开启github pages服务。

![](https://mg0324.github.io/images/github-pages-1.png)

![](https://mg0324.github.io/images/github-pages-2.png)

![](https://mg0324.github.io/images/github-pages-3.png)

![](https://mg0324.github.io/images/github-pages-4.png)

发布完成后，访问 <a href="https://mg0324.github.io/mgwiki" target="_blank">https://mg0324.github.io/mgwiki</a>,不是404，则说明github pages服务启动成功。

开启github pages成功后，会生成gh-pages的分支，如下图：
![](/images/github-pages-5.png)

* 2.3.git clone 远程仓库上的gh-pages分支，github pages服务器访问的就是gh-pages上的文件，默认是index.html。
所以接下来，我们就只需要修改mgwiki上的gh-pages分支的文件资源，这样就能够从<a href="https://mg0324.github.io/mgwiki" target="_blank">https://mg0324.github.io/mgwiki</a>看到不同的效果了。
可参考：<a href="http://www.cnblogs.com/lijiayi/p/githubpages.html" target="_blank">http://www.cnblogs.com/lijiayi/p/githubpages.html</a>

* 2.4.去 <a href="https://github.com/TevinLi/amWiki" target="_blank">amWiki</a> 下载amwiki的项目，
按照amwiki的安装步骤得到目录结构，copy到mgwiki下，得到目录图为：

<img src="https://mg0324.github.io/images/mgwiki-folder.png" style="width:300px;margin-left:20px;"/>

* 2.5.将修改后的gh-pages上的文件，提交并上传到github上的mgwiki的gh-pages分支上。

命令分别是：`git add .` , `git commit -m'msg'` , `git push origin gh-pages` 。

实际日志：


	meigangdeMacBook-Pro:mgwiki meigang$ git add .
	meigangdeMacBook-Pro:mgwiki meigang$ git commit -m"添加github pages建立静态站点"
	[gh-pages 019567c] 添加github pages建立静态站点
	 9 files changed, 55 insertions(+), 2 deletions(-)
	 create mode 100644 images/github-pages-1.png
	 create mode 100644 images/github-pages-2.png
	 create mode 100644 images/github-pages-3.png
	 create mode 100644 images/github-pages-4.png
	 create mode 100644 images/github-pages-5.png
	 create mode 100644 images/mgwiki-folder.png
	 create mode 100644 "library/005-github\347\233\270\345\205\263/001-github pages\346\220\255\345\273\272\351\235\231\346\200\201\345\234\250\347\272\277\344\270\252\344\272\272\347\253\231\347\202\271.md"
	meigangdeMacBook-Pro:mgwiki meigang$ git push origin gh-pages
	Counting objects: 14, done.
	Delta compression using up to 4 threads.
	Compressing objects: 100% (14/14), done.
	Writing objects: 100% (14/14), 1.24 MiB | 0 bytes/s, done.
	Total 14 (delta 4), reused 0 (delta 0)
	remote: Resolving deltas: 100% (4/4), completed with 4 local objects.
	To git@github.com:mg0324/mgwiki.git
	   182790c..019567c  gh-pages -> gh-pages
	meigangdeMacBook-Pro:mgwiki meigang$ 

### 3.外网静态轻文档访问

![](https://mg0324.github.io/images/github-pages-6.png)

恭喜，您拥有了自己的文档站点。
