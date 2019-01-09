---
title: centOS6.7下搭建JAVAWeb服务器
tags:
categories:
---
1. 安装java，通过yum安装想要的版本的java，需要注意的是java版本和tomcat的版本要匹配。[官网链接](http://tomcat.apache.org/whichversion.html)

2. 安装tomcat，要匹配java版本。通过yum安装tomcat。
通过yum安装后，[tomcat的位置](https://blog.csdn.net/u012167045/article/details/55272370)

3. 安装tomcat之后，运行tomcat
	

	service start tomcat


4. 查看tomcat的页面，如果为空白页面的话，需要安装相关[插件](https://stackoverflow.com/questions/10189817/apache-tomcat-installed-and-running-but-localhost8080-presents-blank-page-in-b)
	

	sudo yum install tomcat-admin-webapps.noarch tomcat-docs-webapp.noarch tomcat-javadoc.noarch tomcat-systemv.noarch tomcat-webapps.noarch

5. 安装mysql，通过yum安装mysql-server。安装好之后需要远程连接的，有时候不能远程连接，这时候需要创建远程连接账号。


	mysql> CREATE USER 'monty'@'%' IDENTIFIED BY 'some_pass';
	mysql> GRANT ALL PRIVILEGES ON *.* TO 'monty'@'%'
	    ->     WITH GRANT OPTION;

其中的some_pass需要你自己替换成你想要的密码。

