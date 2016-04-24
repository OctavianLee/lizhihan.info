title: 在Debian系统下完成PostgreSQL环境搭建
date: 2015-04-04 11:42:35
tags: [PostgreSQL,系统服务,Debian]
categories: 系统服务
---

基于项目要求，可能会在新的项目中使用PostgreSQL来构建数据库，如何初步搭建一个PostgreSQL，本篇将简要讲解这个数据库系统的环境搭建。

# 安装PostgreSQL

在Debian系统中，安装postgresql可以通过默认的包管理器进行安装：

``` shell
$ sudo apt-get -y install postgresql-client
$ sudo apt-get -y install postgresql
```
这样在提示完成后便可以完成PostgreSQL的安装，下面要对它进行配置以让我们可以使用它进行开发使用。

# 配置PostgreSQL
## 创建使用用户

默认情况下，PostgreSQL会创建一个名为postgres的特权用户，但是我们真正使用的时候不是通过它来进行的，我们需要利用postgres用户创建我们特定的使用用户。

首先，利用管理员权限切换成postgres用户：

``` shell
$ sudo su postgres
```
在输入你的root的密码之后，你将成功切换成postgres用户，下面你可以通过PostgreaSQL自带的命令来创建超级用户了：

``` shell
$ createuser --superuser postroot
```
这里的postroot是超级用户的名字，这样就建立了这个特权用户，为了让它能够被使用下面，需要为这个用户设置初始密码，登录PostgreSQLs数据库：
``` shell
$ psql
```
然后使用下面命令完成对初始密码的构建：

``` 
# \password postroot
```
这样我们便成功利用PostgreSQL的默认用户创建了我们使用的超级用户了。

## 创建开发使用数据库
在完成用户创建之后，我们开发过程中需要创建一个可以被我们使用的数据库，下面我们便要创建它，首先我们退出我们连上的PostgresSQL数据库：

```
# \q
```
通过如下命令创建一个使用者为postroot的名为test的数据库：

``` shell
$ createdb -O postroot test
```
这样我们便完成了数据库的创建，下面测试一下是否能够成功登录到test数据：

``` shell
$ psql -U postroot -d test -W
```
正确输入密码后没有提示错误，这样我们便成功的完成PostgreSQL的环境配置了。

## 错误提示

在配置过程完成，登录数据库的时候，还是发生了错误，错误信息如下：

``` shell
psql: FATAL:  Peer authentication failed for user "postroot"
```
这里，你需要切换到正常的用户，然后修改/etc/postgresql/x.x/main/pg_hba.conf的文件，找到<code>local all all peer</code>后修改成<code>local all all md5</code>
接着输入下面命令：

``` shell
$ sudo service postgresql reload
```
重新载入PostgreSQL的配置文件，然后继续用postgres用户进行登录这样便可以成功登录了。

以上便是关于在Debian系统下进行PostgreSQL环境搭建的全部内容了。

***

版本修改记录：

+ 2014-11-17 完成本次文章全部内容 By Octavian
+ 2015-04-04 博客版本发布 By Octavian
+ 2015-05-27 格式修改 By Octavian
+ 2016-04-06 更正文档中命令错误 By Octavian