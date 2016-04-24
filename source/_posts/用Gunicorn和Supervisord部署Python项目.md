title: 用Gunicorn + Supervisord部署Python项目
date: 2015-03-01 23:35:03
tags: [Python,系统服务,Gunicorn,Supervisord]
categories: 系统服务
---

当开发完成一个Python项目，为了让项目成功在服务器上部署，我们需要一些工具来完成，当然选择的种类繁多，本文使用的是Gunicorn + Supervisord的方式来进行操作。

本文将结合官方文档来解释并介绍如何配置Gunicorn和Supervisord

# 什么是Gunicorn和Supervisord

Gunicorn是一个Python WSGI的HTTP服务器，它可以兼容多种Web框架，实现简单，服务器资源轻量且速度快。

Supervisord是一个可以让用户监控进程的C/S系统。

简单来说，Gunicorn是来启动Python项目的，而Supervisord是来监控Python项目的。

# 如何使用Gunicorn和Supervisord

知道了这两个工具是做什么的之后，下面我们来进行相关的配置，首先我们将从Gunicorn开始。

## Gunicorn的使用

### 安装Gunicorn
Gunicorn的安装需要在虚拟环境下进行，在虚拟环境启动后使用pip工具安装最新版本的Gunicorn，命令如下：

<code>$ pip install gunicorn</code>

这样你就成功地安装了Gunicorn了。

### 安装gevent
这里我们需要安装Gevent来继续进行我们的安装配置，命令如下：

<code>$ pip install gevent</code>

####为什么要装gevent
有时候，我们需要进行一些异步的操作，gevent是能够良好的帮我们去实现这样的需求！

### 填写Gunicorn配置文件
首先新建一个Gunicorn的配置文件，例如名为gunicorn.py，命令如下：

<code>$ touch gunicorn.conf</code>

编辑该文档：

<code>$ vim gunicorn.conf</code>

如果我们需要配置一个名为myproject的项目的启动，配置如下内容：

```
import gevent.monkey
gevent.monkey.patch_all()

import multiprocessing

bind = 'unix:/home/myproject/var/run/myproject.sock'
user = 'myproject'
worker_class = 'gevent'
workers = multiprocessing.cpu_count() * 2 + 1
```

####配置解释
```python
import gevent.monkey
gevent.monkey.patch_all()
```

这里导入gevent的猴子补丁（注：可以让一些不能与gevent共同工作的库进行协同工作，具体内容这里不做介绍）

<code>bind = 'unix:/home/myproject/var/run/myproject.sock'</code>
将myproject项目的地址进行绑定，这里要在nginx服务器配置中预先指明。

<code>user = 'myproject'</code>
用名为myproject的用户的操作worker进程，需要预先为该项目配置添加该用户。

<code>worker_class = 'gevent'</code>
指明worker的种类，这里我们使用gevent来处理，所以是gevent。

```
import multiprocessing
workers = multiprocessing.cpu_count() * 2 + 1
```

指明workers的数目，官方推荐的最优的数量为cpu数量＊2+1。

以上便是配置文件内容具体含义的解释，更多的设置等参见官方文档：<link>http://docs.gunicorn.org/en/latest/settings.html</link>

### 启动Gunicorn

启动Gunicorn是通过Supervisord来进行的，但是目前可以通过gunicorn的命令来运行，命令如下：

<code>$ gunicorn -c gunicorn.conf [文件名]:[调用的模块名]</code>


## Supervisord的使用

### 安装Supervisord
下面我们需要安装supervisord，在系统环境下输入如下命令完成安装：

<code>$ sudo pip install supervisor</code>

### 生成supervisord配置文件
使用echo_supervisord_conf来生成配置文件，例如生成一个名为supervisord.conf的文件，命令如下：
```
$ echo_supervisord_conf supervisor.conf
```
或者自己生成一个空白的文件，命令如下：

<code>$ touch supervisor.conf</code>

编辑该文档：

<code>$ vim supervisor.conf</code>

同样我们需要监视一个名为myproject的项目的启动，配置如下内容：
```
[program:myproject]
command=gunicorn [文件名]:[模块名称] -c gunicorn.conf
directory=/home/myproject/
autostart=true
redirect_stderr=true

stdout_logfile=/home/myproject/var/logs/gunicorn-out.log
stderr_logfile=/home/myproject/var/logs/gunicorn-err.log
```

#### 配置解释
```
[program:myproject]
```
设置程序名称，为myproject。
```
command=gunicorn [文件名]:[模块名称] -c gunicorn.conf
```
启动gunicorn的命令。
```
autostart=true
```
设置当supervisord启动，程序会自动运行。
```
redirect_stderr=true
```
设置如果有错误会将错误输出填入错误输出文件内。
```
stdout_logfile=/home/myproject/var/logs/gunicorn-out.log
stderr_logfile=/home/myproject/var/logs/gunicorn-err.log
```
指明具体输出文件的名称位置。

以上便是配置文件内容具体含义的解释，更多的设置等参见官方文档：<link>http://supervisord.org/configuration.html</link>

### 启动supervisord
输入如下命令启动该项目的supervisord:

<code>$ supervisord -c supervisord.conf</code>

注：由于我们线上的测试服务器全局已经安装好了supervisord，所以在项目下新建的supervisord文件只需导入到运行的系统配置下即可，在/etc/supervisord的末尾添加新的supervirsord的配置文件路径即可。

以上便是用Gunicorn + Supervisord部署Python项目的具体内容。

***

版本修改记录：

+  2014-11-10 完成本次文章全部内容 By Octavian
+  2015-03-01 修改章节中的文本说明 By Octavian
+  2015-03-17 发布博客版本 By Octavian




