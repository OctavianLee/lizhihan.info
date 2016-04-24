title: Memcached的简单使用
date: 2015-05-27 10:11:20
tags: [Memcached,系统服务,Python,缓存,Pylibmc]
categories: 系统服务
---

本文将针对Memcached进行介绍并说明它的一些简单的使用。

# 什么是Memcached

## 实际中的问题
在Web应用中，数据通常都存储在数据库中，在需要数据的时候对数据发送请求。但是都知道读取数据的过程是消耗资源的，当请求量变大的时候便会使Web应用的性能下降。解决这类问题的有效方法，便是添加对数据库的缓存，这样能减少数据库访问次数，降低开销。

而memcached便是实际中会使用到的缓存服务器。
![Memcached](http://7xjc5f.com1.z0.glb.clouddn.com/img/memcached.png)

## Memcached介绍
Memcached是C/S架构的，其特点如下：

+ 协议简单
+ 基于libevent的事件处理
+ 内置内存存储方式
+ memcached不互相通信的分布式


###服务器端
服务器端维护一个键值关联数组并通过计算键的hash来决定存储在哪里和读取哪一个数值。

这些值存放在内存中，在如下情况下会发生丢弃：

+ 为缓存分配的内存耗尽
+ 条目被明确删除
+ 条目过期失效

此外服务器之间彼此不知道其他服务器信息，不互相通信。

###客户端
客户端填充数组并进行查询，键的长度达到250bytes，值的大小最多有1M。

客户端知道所有服务器信息。当客户通过特定的键设置或者查询值的时候，会计算键的hash来决定服务器去使用。

# Memcached的使用

##服务器的安装
关于服务器方面的安装这里不做过多介绍，可以参考官网
http://memcached.org/downloads 或者其他的资料进行了解。

##客户端的使用

基于Python的memcached客户端库有很多，比较著名的就是python-memcached，但这边介绍的是一个效率更出色的库pylibmc，在面对大量数据的读写时能提升很多速率。（参见：http://kzooeny.blogspot.jp/2013/05/compare-pylibmc-and-python-memcached.html）

### 关于pylibmc
pylibmc的接口与python-memcached相似，便于替换，它是封装了libmemcached库的一个客户端库。（关于libmemcached的特点参见：http://docs.libmemcached.org/libmemcached.html ）

### pylibmc基本的使用

#### 安装pylibmc
pylibmc安装非常容易，通过pip即可：

```python
pip install pylibmc
```
当然也可以通过setup进行安装（参见：http://sendapatch.se/projects/pylibmc/install.html ）

#### 启动pylibmc
安装完成后便能正常启动pylibmc了：

```python
import pylibmc
mc = pylibmc.Client(["127.0.0.1"], binary=True,
    behaviors={"tcp_nodelay": True,
        "verify_keys": True})
        "ketama": True})
```

其中behaviors的字段提供了pylibmc的一些使用策略（参见：http://sendapatch.se/projects/pylibmc/behaviors.html ）

#### pylibmc操作
pylibmc的操作非常简单，常见的操作说明如下：

1 设置操作

```python
mc['key1'] = "value"
mc.set('key2', "value")
```

2 查询操作

```python
mc['key1']
mc.get('key2')
```

3 删除操作

```python
del mc['key1']
mc.delete('key2')
```

其他诸如增减操作等参见官方文档：http://sendapatch.se/projects/pylibmc/


以上便是对memcached的简单介绍和使用说明。

***

版本修改记录：

+ 2015-05-29 完成本篇完成全部内容 By Octavian