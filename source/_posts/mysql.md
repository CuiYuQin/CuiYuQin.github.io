---
title: mysql踩坑记录
date: 2020-08-26 22:02:10
tags: ['技术']
categories: 技术
id: mysql
---

事情起因是这样的，我最近想着写个用python爬取某网站音频并写入数据库的脚本，由于之前操作数据库都是在线上直接改，没有本地连过，所以本地的pymysql想连我网站的数据库，第一步是要把user表的localhost改成%，以此开放远程连接，问题就来了，我在phpmyadmin改完之后，居然所有账号都登不上了，也不知道哪出了问题，可能是我敲错了吧。然后我又找了一篇新建root用户的博文，跟着敲，敲完之后确实能登上去了。但是！这时候只能看到information_schema这一个数据库了，这时候网站也挂了，我以为数据库被误删了，就找各种方法想还原数据库。折腾半天，我终于解决了，我觉得有必要记录一下踩坑过程。

<!-- more -->

**只能看到information_schema的原因：用户没权限，而不是数据库被误删了。**

## 思路

按下面的解决步骤执行到第4步，如果执行 `show databases;`命令之后，数据库都显示出来了，那么就是root用户没给够权限，按下面的步骤走完，重启mysql就行了。

1. 找到 `/etc/my.conf` 文件，在 `[mysqld]` 一行下面加上 `skip-grant-tables`，跳过权限验证。
2. 然后运行 `service mysqld restart` 重启一下`mysql`，注意是`mysqld`不是`mysql`。
3. 运行 `mysql -u root -p` 登录root用户。
4. 登录成功后可以用 `show databases;`命令 看看所有的数据库是否都显示出来了。注意记得加上`;` 
5. 执行：`use mysql;` 这时候显示 `Database changed`
6. 执行：`flush privileges;`
7. 执行：`grant all privileges on *.* to root@'localhost' identified by "a123456" with grant option; `，这一步是很重要的一步，就是为了把所有权限授予给root用户
8. 去掉第一步 `/etc/my.conf` 文件里的`skip-grant-tables`
9. 执行：`service mysqld restart`，重启mysql

最后查看root用户权限

```sql
select * from mysql.user where user='root'\G;
```
可以看到都显示是Y，这就成功了。

## localhost受限制
如果以上用 `mysql -u root -p` 无法登录，先保证密码是对的，然后尝试用127.0.0.1登录，执行 `mysql -u root -p 数据库 -h 127.0.0.1` ，如果可以登录，那么就是localhost受限制。执行以上第7步就行了。

**下面解释上面第7步赋予权限命令的含义**


all privileges：表示将所有权限授予给用户。也可指定具体的权限，如：SELECT、CREATE、DROP等。

on：表示这些权限对哪些数据库和表生效，格式：数据库名.表名，这里写“*”表示所有数据库，所有表。如果我要指定将权限应用到test库的user表中，可以这么写：test.user

to：将权限授予哪个用户。格式：”用户名”@”登录IP或域名”。%表示没有限制，在任何主机都可以登录。比如：”root”@”192.168.0.%”，表示root这个用户只能在192.168.0IP段登录

identified by：指定用户的登录密码

with grant option：表示允许用户将自己的权限授权给其它用户

root用户可以登录，其他用户不行

把其他user也按照上面同样的命令开放权限

## 拓展知识

mysql 是个命令行程序；mysqld 是服务。linux 系统里一般的服务都是以 d 结尾的，比如 httpd，vsftpd 等等。d 的全拼应该是 daemon，也就是守护程序的意思，常驻于后台。

千万要定期备份数据库！显示不出数据库别慌张，很大概率是用户权限的问题，如果真的不小心把数据库删了，可以通过 `mysqlbinlog` 命令将二进制日志文件恢复到数据库。

另外想说，网上很多博文真的是误人子弟，博客园csdn一大堆复制粘贴，最后连命令都没粘贴对，对于不懂数据库linux操作的，还是要学习一下基本知识。
