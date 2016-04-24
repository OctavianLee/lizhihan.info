title: Redis事务
date: 2015-09-14 22:37:12
tags: [Redis, transanction, 事务]
categories: 系统服务
---

本文对Redis事务做一个基本的介绍。

# 使用背景
在使用Redis时，可以认为每一个操作都是原子级，在实际过程中经常会遇到多个命令一起执行的情况。

这种情况下，这些命令通常相互关联，要么全部被执行，要么都不被执行。

Redis使用事务(Transaction)来保证命令的独立以及多个命令一起执行。

# 事务相关命令

Redis事务的基本命令包含：

+ MULTI；
+ EXEC；
+ DISCARD；
+ WATCH。

## MULTI & EXEC

`MULTI`命令用于开启一个事务，当执行后，继续的多条命令将会添加到队列(QUEUE)中，不会立即执行，当EXEC命令执行时，队列中的命令才能被执行。

`EXEC`命令负责触发并执行事务中的所有命令：

+ 如果开启一个事务，未能成功执行`EXEC`，事务中所有命令都不会执行。
+ 如果开启事务之后成功执行`EXEC`，事务中的所有命令都会被执行。


用法案例：

```
> MULTI
OK
> SET key 1
QUEUED
> INCR key
QUEUED
> EXEC
1) (integer) 1
2) (integer) 2
```

案例说明：

+ `MULTI`成功创建事务时，总是返回`OK`；
+ 当传入命令时，会发出`QUEUED`的状态回复(status reply)表明加入队列，这些命令将在`EXEC`时执行；
+ `EXEC`的回复是一个数组，回复的按照命令的先后进行。


## DISCARD

`DISCARD`可以放弃事务，让事务队列清空。

用法案例：

```
> SET key 1
OK
> MULTI
OK
> INCR key
QUEUED
> DISCARD
OK
> GET key
"1"
```

## WATCH

`WATCH`可以为Redis事务提供check-and-set(CAS)。

`WATCH`监控的键只有没有被修改时，事务才能执行，否则整个事务都会被取消，`EXEC`会返回空多条批量回复(null multi-bulk reply)来表示事务已经失败，这种锁的行为被称为乐观锁。

`WATCH`的键可以通过`UNWATCH`命令手动来取消。

注意点：

+ `WATCH`监视了一个带过期时间的键，当键过期了，事务仍然可以正常执行；
+ 当`EXEC`被调用，不管事务是否成功执行，对所有键的监视都会被取消；
+ 当客户端断开连接时，该客户端对键的监视会被取消。


# 事务错误

在执行`EXEC`之前，进入队列的命令出错，服务器会对命令入队失败的情况进行记录，并在调用`EXEC`命令时，拒绝执行并自动放弃这个事务。

在执行`EXEC`后出错，事务中的其他命令仍然会继续执行，不进行回滚(roll back)。

## 不支持回滚

Redis命令只会因为语法和错误使用报错，对于执行逻辑中产生的错误，不会被回滚，需要在开发层面上进行纠正。

因为不需要对回滚进行支持，Redis使用上更加简单、灵活、高效。


***

版本修改记录：

+ 2015-09-15 完成本次文章全部内容。 By Octavian