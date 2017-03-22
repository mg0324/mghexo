---
title: 'mysql读写分离,中间件用mycat'
date: 2016-11-04 16:01:17
categories: mysql
tags:
- mysql
- mycat
- 性能
---


	首先，数据库的读写分离，能让应用对数据库的访问压力下降，较之一台数据库服务器来读写的时候。
	2台数据库服务器，1台用来执行写操作，1台用来执行读操作，这样能够分散应用对数据的压力，而且能加强数据库的数据安全性。
	所以，面对比较大型的数据读取应用，对其数据库做读写分离，对性能提升是很有好处的。

### 为什么读写分离可以提高性能？

	
	1.物理服务器增加，负荷增加
	2.主从只负责各自的写和读，极大程度的缓解X锁和S锁争用
	3.从库可配置myisam引擎，提升查询性能以及节约系统开销
	4.至于你提到的“master所执行的（写）的所有语句，都会在slave被执行一遍”这个只说对一半，从库同步主库的数据和主库直接写还是有区别的，通过主库发送来的binlog恢复数据，但是，最重要区别在于主库向从库发送binlog是异步的，从库恢复数据也是异步的。
	5.读写分离适用与读远大于写的场景，如果只有一台服务器，当select很多时，update和delete会被这些select访问中的数据堵塞，等待select结束，并发性能不高。 对于写和读比例相近的应用，应该部署双主相互复制。

### 主要步骤
* 1.mysql主从数据库设置，master用来做写操作，slave用来做读操作。
> 主从数据库设置，是指slave会通过mysql的主从复制功能，自动去实时去同步master上的数据变更。（单向的slave copy master）

* 2.mycat设置简单的读写分离，并启动，对应用程序端提供mysql porxy代理服务。
> mycat是一款国人开发的数据库代理中间件，功能强大，可以支持多种数据库，支持分库分表。

### 实际操作
* 1.搭建mysql的master-slave环境
> 有2台MySQL数据库主机，IP分别为192.168.1.113 和 192.168.1.109。<br/>
 把109做master，用来写入数据，执行insert,update,delete等操作。<br/>
 把113做slave，用来读取数据，执行select查询操作。
* 1.1.配置master

在mysql的配置文件中my.ini的[mysqld]节点下加上如下配置：


	log-bin=mysql-bin #slave会基于此log-bin来做replication
	server-id = 1 #master的标示
	binlog-do-db = jpress #用于master-slave的具体数据库

然后给复制集配置复制的用户，执行如下命令。


 	然后添加专门用于replication的用户：
	mysql> GRANT REPLICATION SLAVE ON *.* TO xiaogang@192.168.1.113 IDENTIFIED BY '123456';

重启mysql服务，然后查看show master status;

	
	mysql> show master status;
	+------------------+----------+--------------+------------------+
	| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB |
	+------------------+----------+--------------+------------------+
	| mysql-bin.000008 |      107 | jpress       |                  |
	+------------------+----------+--------------+------------------+
	1 row in set (0.00 sec)
* 1.2.配置slave

<font style="color:red;">补上遗漏的部分：2016-12-22</font>
slave上的my.ini的[mysqld]节点下加上如下配置：
	

	server-id = 2 #slave的标示,推荐使用IP的第4部分
	#其他配置可以百度mysql主从复制详细配置
	#relay-log-index=slave-relay-bin.index
	#relay-log=slave-relay-bin

如果有配置主从，要先关闭`stop slave;`，然后再执行如下命令，来配置主从设置：


	CHANGE MASTER TO   
	MASTER_HOST='192.168.1.109',   
	MASTER_USER='xiaogang',   
	MASTER_PASSWORD='123456',   
	MASTER_LOG_FILE='mysql-bin.000008',   
	MASTER_LOG_POS= 107;

其中用户就是master数据库服务器上分配的用户，file和pos是master上的`show master status;`中的file和position。
执行完之后，启动主从关联，`slave start;`，然后使用`show slave status;\G`来查看从服务器状态。

	
	mysql> show slave status\G;
	*************************** 1. row ***************************
	               Slave_IO_State: Waiting for master to send event
	                  Master_Host: 192.168.1.109
	                  Master_User: xiaogang
	                  Master_Port: 3306
	                Connect_Retry: 60
	              Master_Log_File: mysql-bin.000008
	          Read_Master_Log_Pos: 107
	               Relay_Log_File: PC-20150410VNLS-relay-bin.000008
	                Relay_Log_Pos: 253
	        Relay_Master_Log_File: mysql-bin.000008
	             Slave_IO_Running: Yes
	            Slave_SQL_Running: Yes
	              Replicate_Do_DB: 
	          Replicate_Ignore_DB: 
	           Replicate_Do_Table: 
	       Replicate_Ignore_Table: 
	      Replicate_Wild_Do_Table: 
	  Replicate_Wild_Ignore_Table: 
	                   Last_Errno: 0
	                   Last_Error: 
	                 Skip_Counter: 0
	          Exec_Master_Log_Pos: 107
	              Relay_Log_Space: 565
	              Until_Condition: None
	               Until_Log_File: 
	                Until_Log_Pos: 0
	           Master_SSL_Allowed: No
	           Master_SSL_CA_File: 
	           Master_SSL_CA_Path: 
	              Master_SSL_Cert: 
	            Master_SSL_Cipher: 
	               Master_SSL_Key: 
	        Seconds_Behind_Master: 0
	Master_SSL_Verify_Server_Cert: No
	                Last_IO_Errno: 0
	                Last_IO_Error: 
	               Last_SQL_Errno: 0
	               Last_SQL_Error: 
	  Replicate_Ignore_Server_Ids: 
	             Master_Server_Id: 1
	1 row in set (0.01 sec)

其中
	

	Slave_IO_Running: Yes
	Slave_SQL_Running: Yes
都是yes则说明主从配置已经生效。{意味着在master数据库jpress上的操作都会被同步到slave上}。

如是，MySQL的主从复制就已经搭建完毕，注意是从 copy 主，是单向复制的，也就是说write得指定是master。

* 2.基于mycat来搭建读写分离
* 2.1.先下载mycat
> 具体了解，请参考mycat官网：http://www.mycat.org.cn/<br/>
到https://github.com/MyCATApache/Mycat-download/tree/master/1.6-RELEASE 下载最新版本1.6，根据自己的OS来。
下载后的目录如下（mac平台下）：
![](http://mg0324.github.io/images/mycat-floder.png)

* 2.2.简单配置mycat的读写分离

server.xml


	<!-- 配置mycat访问用户，对应用程序，也就是master数据库的用户，其中的数据库名称是指暴露给jdbc的名称，可以随便取，然后 -->
	<user name="root">
		<property name="password">meigang2016</property>
		<property name="schemas">testdb</property>
	</user>

schema.xml

	<?xml version="1.0"?>
	<!DOCTYPE mycat:schema SYSTEM "schema.dtd">
	<mycat:schema xmlns:mycat="http://io.mycat/">
		<!-- 不设置分库，分片，只需要一个节点。dataNode指向下面配置的 -->
		<schema name="testdb" checkSQLschema="false" sqlMaxLimit="100" dataNode="dn1">
			
		</schema>
		<!-- 其中database指向的是真的数据库名称 -->
		<dataNode name="dn1" dataHost="localhost1" database="jpress" />
		
		<dataHost name="localhost1" maxCon="1000" minCon="10" balance="1"
				  writeType="0" dbType="mysql" dbDriver="native" switchType="1"  slaveThreshold="100">
			<!-- 心跳测试 -->
			<heartbeat>show slave status</heartbeat>
			<!-- can have multi write hosts，localhost - 192.168.1.109 -->
			<writeHost host="hostM1" url="localhost:3306" user="root"
					   password="meigang2016">
				<!-- can have multi read hosts -->
				<readHost host="hostS2" url="192.168.1.113:3306" user="xiaogang" password="123456" />
			</writeHost>
		</dataHost>
	</mycat:schema>

* 2.3.启动mycat

	
	unix/linux 启动命令
	meigangdeMacBook-Pro:bin meigang$ ./mycat start
	Starting Mycat-server...

查看启动日志，表示启动成功。

	INFO   | jvm 1    | 2016/10/18 15:46:16 | MyCAT Server startup successfully. see logs in logs/mycat.log
	INFO   | jvm 1    | 2016/10/18 15:46:17 | 2016-10-18 15:46:16,991 [INFO ][$_NIOREACTOR-1-RW] connectionAcquired MySQLConnection [id=13, lastTime=1476776776969, user=xiaogang, schema=jpress, old shema=jpress, borrowed=false, fromSlaveDB=true, threadId=14, charset=utf8, txIsolation=3, autocommit=true, attachment=null, respHandler=null, host=192.168.1.113, port=3306, statusSync=null, writeQueue=0, modifiedSQLExecuted=false]  (io.mycat.backend.mysql.nio.handler.NewConnectionRespHandler:NewConnectionRespHandler.java:45)

使用mysql命令行工具，登录8066端口，验证mycat启动数据库代理成功。

	
	meigangdeMacBook-Pro:bin meigang$ mysql -P8066 -uroot -p
	Enter password: 
	Welcome to the MySQL monitor.  Commands end with ; or \g.
	Your MySQL connection id is 27
	Server version: 5.5.50-log MySQL Community Server (GPL)

	Copyright (c) 2000, 2016, Oracle and/or its affiliates. All rights reserved.

	Oracle is a registered trademark of Oracle Corporation and/or its
	affiliates. Other names may be trademarks of their respective
	owners.

	Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

	mysql> 

* 3.使用jpress开源Java博客系统来链接8066的testdb数据库服务，做验证。

将jpress压缩成war后放到tomcat的webapps中，运行后得到解压后的项目，把war删掉。
进去找到/web-inf/classes/db.properties，修改如下：

	
	#Auto create by JPress
	#Mon Oct 17 18:54:30 CST 2016
	db_name=testdb
	db_host_port=8066
	db_tablePrefix=jpress_
	db_host=localhost
	db_password=meigang2016
	db_user=root

启动tomcat，查看master和slave上的执行sql的日志。可以发现master上执行的多是update,delete,insert的，而slave上执行的
多是select（有ddl的，应该是slave去同步master上执行的）

MySQL启用日志：
set global general_log = on;

show variables like 'general_log_file';去查看该文件，就可以查看该数据库上执行的sql.
	
如下是master 109上执行的日志。都是ddl的。


	Time                 Id Command    Argument
	161018 16:16:35	   28 Query	show variables like 'general_log_file'
	161018 16:16:36	   22 Query	show slave status
	161018 16:16:41	   28 Query	set global general_log = on
	161018 16:16:45	   28 Query	show variables like 'general_log_file'
	161018 16:16:46	   20 Query	show slave status
	161018 16:16:56	   26 Query	show slave status
	161018 16:17:06	   23 Query	show slave status
	161018 16:17:11	   21 Query	SET names utf8;update `jpress_content` set `view_count` = 1  where `id` = '4'
	161018 16:17:16	   25 Query	show slave status
	161018 16:17:25	   17 Query	SET names utf8;update `jpress_content` set `view_count` = 5  where `id` = '3'
	161018 16:17:26	   19 Query	show slave status
	161018 16:17:36	   18 Query	show slave status
	161018 16:17:46	   24 Query	show slave status
	161018 16:17:56	   22 Query	show slave status
	161018 16:18:06	   20 Query	show slave status
	161018 16:18:09	   26 Query	SET autocommit=0;insert into `jpress_content`(`module`, `text`, `status`, `meta_keywords`, `comment_status`, `remarks`, `lng`, `modified`, `id`, `title`, `thumbnail`, `flag`, `meta_description`, `created`, `user_id`, `slug`, `lat`) values('page', '<p>测试</p>', 'normal', null, null, null, null, '2016-10-18 16:18:09', null, '测试', null, null, null, '2016-10-18 16:18:09', '1', '测试', null)
	161018 16:18:10	   26 Query	SELECT COUNT(*) FROM `jpress_comment` WHERE  content_id = '5' and status='normal'
			   26 Query	DELETE FROM `jpress_mapping` WHERE content_id = '5'
			   26 Query	commit
			   26 Query	rollback
			   23 Query	SET names utf8;update `jpress_content` set `text` = '<p>测试</p>'  where `id` = 5
			   21 Query	update `jpress_user` set `content_count` = 3  where `id` = '1'
	161018 16:18:16	   25 Query	show slave status
如下是slave 113上执行的日志。多是select的，还有用来同步数据的sql.
	

	161018 16:22:54	   19 Query	select 1
		   12 Query	select * from `jpress_user` where `id` = '1'
		   13 Query	select * from `jpress_content` c where module = 'menu'  ORDER BY c.order_number ASC
		   16 Query	select count(*)  from `jpress_content` c left join `jpress_mapping` m on c.id = m.`content_id` left join `jpress_taxonomy`  t on  m.`taxonomy_id` = t.id left join `jpress_user` u on c.user_id = u.id WHERE  ( c.module = 'article'  )  AND  c.status = 'normal'  group by c.id
		   17 Query	select c.*,GROUP_CONCAT(t.id ,':',t.slug,':',t.title,':',t.type SEPARATOR ',') as taxonomys,u.username,u.nickname,u.avatar  from `jpress_content` c left join `jpress_mapping` m on c.id = m.`content_id` left join `jpress_taxonomy`  t on  m.`taxonomy_id` = t.id left join `jpress_user` u on c.user_id = u.id WHERE  ( c.module = 'article'  )  AND  c.status = 'normal'  group by c.id ORDER BY c.created DESC limit 0, 10
		   18 Query	select  c.*,GROUP_CONCAT(t.id ,':',t.slug,':',t.title,':',t.type SEPARATOR ',') as taxonomys  ,u.username,u.nickname,u.avatar  from `jpress_content` c left join `jpress_mapping` m on c.id = m.`content_id` left join `jpress_taxonomy`  t on  m.`taxonomy_id` = t.id left join `jpress_user` u on c.user_id = u.id  where c.status = 'normal'  AND  ( c.module = 'article'  )  AND  ( c.flag like '%hot%'  ) GROUP BY c.id ORDER BY c.created DESC LIMIT 0, 3
		   14 Query	select  c.*,GROUP_CONCAT(t.id ,':',t.slug,':',t.title,':',t.type SEPARATOR ',') as taxonomys  ,u.username,u.nickname,u.avatar  from `jpress_content` c left join `jpress_mapping` m on c.id = m.`content_id` left join `jpress_taxonomy`  t on  m.`taxonomy_id` = t.id left join `jpress_user` u on c.user_id = u.id  where c.status = 'normal'  AND  ( c.module = 'article'  )  AND c.thumbnail is not null GROUP BY c.id ORDER BY c.comment_count  DESC  LIMIT 0, 3
		   15 Query	select  c.*,GROUP_CONCAT(t.id ,':',t.slug,':',t.title,':',t.type SEPARATOR ',') as taxonomys  ,u.username,u.nickname,u.avatar  from `jpress_content` c left join `jpress_mapping` m on c.id = m.`content_id` left join `jpress_taxonomy`  t on  m.`taxonomy_id` = t.id left join `jpress_user` u on c.user_id = u.id  where c.status = 'normal'  AND  ( c.module = 'article'  )  AND c.thumbnail is not null GROUP BY c.id ORDER BY c.view_count  DESC  LIMIT 0, 10
		   19 Query	SELECT * FROM `jpress_taxonomy` t WHERE  t.content_module = 'article'  AND  t.`type` = 'tag'  ORDER BY t.created DESC limit 10
	161018 16:22:56	   12 Query	SELECT * FROM `jpress_taxonomy` t WHERE  t.content_module = 'article'  AND  ( t.`slug` = 'mysql'  )
			   13 Query	SELECT * FROM `jpress_content` WHERE module = 'menu' and object_id = '1' order by id desc
			   16 Query	select count(*)  from `jpress_content` c left join `jpress_mapping` m on c.id = m.`content_id` left join `jpress_taxonomy`  t on  m.`taxonomy_id` = t.id left join `jpress_user` u on c.user_id = u.id WHERE c.status = 'normal'  AND  c.module = 'article'  AND exists(select 1 from `jpress_mapping` m where m.`taxonomy_id` in (1) and m.`content_id`=c.id)  group by c.id
			   17 Query	select c.*,GROUP_CONCAT(t.id ,':',t.slug,':',t.title,':',t.type SEPARATOR ',') as taxonomys,u.username,u.nickname,u.avatar  from `jpress_content` c left join `jpress_mapping` m on c.id = m.`content_id` left join `jpress_taxonomy`  t on  m.`taxonomy_id` = t.id left join `jpress_user` u on c.user_id = u.id WHERE c.status = 'normal'  AND  c.module = 'article'  AND exists(select 1 from `jpress_mapping` m where m.`taxonomy_id` in (1) and m.`content_id`=c.id)  group by c.id ORDER BY c.created DESC limit 0, 10
			   18 Query	select  c.*,GROUP_CONCAT(t.id ,':',t.slug,':',t.title,':',t.type SEPARATOR ',') as taxonomys  ,u.username,u.nickname,u.avatar  from `jpress_content` c left join `jpress_mapping` m on c.id = m.`content_id` left join `jpress_taxonomy`  t on  m.`taxonomy_id` = t.id left join `jpress_user` u on c.user_id = u.id  where c.status = 'normal'  AND  ( c.module = 'article'  )  AND c.thumbnail is not null GROUP BY c.id ORDER BY c.comment_count  DESC  LIMIT 0, 3
			   14 Query	select  c.*,GROUP_CONCAT(t.id ,':',t.slug,':',t.title,':',t.type SEPARATOR ',') as taxonomys  ,u.username,u.nickname,u.avatar  from `jpress_content` c left join `jpress_mapping` m on c.id = m.`content_id` left join `jpress_taxonomy`  t on  m.`taxonomy_id` = t.id left join `jpress_user` u on c.user_id = u.id  where c.status = 'normal'  AND  ( c.module = 'article'  )  AND c.thumbnail is not null GROUP BY c.id ORDER BY c.view_count  DESC  LIMIT 0, 10
	161018 16:22:57	   15 Query	show slave status
			   19 Query	select  c.*,GROUP_CONCAT(t.id ,':',t.slug,':',t.title,':',t.type SEPARATOR ',') as taxonomys  ,u.username,u.nickname,u.avatar  from `jpress_content` c left join `jpress_mapping` m on c.id = m.`content_id` left join `jpress_taxonomy`  t on  m.`taxonomy_id` = t.id left join `jpress_user` u on c.user_id = u.id  WHERE c.id = '3' GROUP BY c.id
			   21 Connect	xiaogang@MACBOOKPRO-E61D on jpress
			    2 Query	BEGIN
			   12 Query	select t.* from `jpress_taxonomy` t left join `jpress_mapping` m on t.id = m.taxonomy_id  where m.content_id = '3'
			    2 Query	update `jpress_content` set `view_count` = 6  where `id` = '3'
			    2 Query	COMMIT /* implicit, from Xid_log_event */
			   13 Query	select count(*)   from `jpress_comment` c left join `jpress_content` on c.content_id = `jpress_content`.id left join `jpress_user` u on c.user_id = u.id  left join `jpress_comment` quote_comment on c.parent_id = quote_comment.id left join `jpress_user` quote_user on quote_comment.user_id = quote_user.id  WHERE   c.`status` = 'normal'  AND   `jpress_content`.id = '3'
			   16 Query	select c.*,`jpress_content`.title content_title,u.username,u.nickname,u.avatar, quote_comment.text qc_content,quote_comment.author qc_author,quote_user.username qc_username,quote_user.nickname qc_nickname,quote_user.avatar qc_avatar    from `jpress_comment` c left join `jpress_content` on c.content_id = `jpress_content`.id left join `jpress_user` u on c.user_id = u.id  left join `jpress_comment` quote_comment on c.parent_id = quote_comment.id left join `jpress_user` quote_user on quote_comment.user_id = quote_user.id  WHERE   c.`status` = 'normal'  AND   `jpress_content`.id = '3' order by c.created desc limit 0, 10
			   17 Query	select  c.*,GROUP_CONCAT(t.id ,':',t.slug,':',t.title,':',t.type SEPARATOR ',') as taxonomys  ,u.username,u.nickname,u.avatar  from `jpress_content` c left join `jpress_mapping` m on c.id = m.`content_id` left join `jpress_taxonomy`  t on  m.`taxonomy_id` = t.id left join `jpress_user` u on c.user_id = u.id  where c.status = 'normal'  AND  ( c.module = 'article'  )  AND c.thumbnail is not null GROUP BY c.id ORDER BY c.comment_count  DESC  LIMIT 0, 3
			   18 Query	select  c.*,GROUP_CONCAT(t.id ,':',t.slug,':',t.title,':',t.type SEPARATOR ',') as taxonomys  ,u.username,u.nickname,u.avatar  from `jpress_content` c left join `jpress_mapping` m on c.id = m.`content_id` left join `jpress_taxonomy`  t on  m.`taxonomy_id` = t.id left join `jpress_user` u on c.user_id = u.id  where c.status = 'normal'  AND  ( c.module = 'article'  )  AND c.thumbnail is not null GROUP BY c.id ORDER BY c.view_count  DESC  LIMIT 0, 10
	161018 16:23:07	   14 Query	show slave status
	161018 16:23:17	   15 Query	show slave status
	161018 16:23:27	   19 Query	show slave status
	161018 16:23:37	   21 Query	show slave status


* 4.查看博客效果
![](http://mg0324.github.io/images/jpress-on-master-slave.png)

