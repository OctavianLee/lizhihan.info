title: Git Stash
date: 2015-07-22 18:15:57
tags: [Git, Stash]
categories: Git
---



# Stash介绍

## 实际场景

在使用Git开发时，开发者在自己的分支上进行开发，由于需要紧急开发一个新的分支，而当前手里的完成的工作还不能合并。

此时，使用`git stash`命令可以将当前做过的修改从工作区保存到暂存区中。

## 常用命令

+ 将当前的任务存到暂存区中：

命令如下：

```shell
$ git stash
```

如果需要，对当前工作进行说明，则命令为：

```shell
$ git stash save 'Message Here'
```

+ 查看暂存列表：

命令如下：

```shell
$ git stash list
```

会得到所有暂存的列表，类似下面格式；

```
stash@{x}: XXXX
```

+ 恢复暂存区内容进度：

命令如下：

```shell
$ git stash pop stash@{x}
```

或者：

```shell
$ git stash apply stash@{x}
```

+ 删除某个进度：

命令如下：

```shell
$ git stash drop stash@{x}
```

+ 清空所有暂存区进度：

命令如下：

```shell
$ git stash clear
```

# 其他用法说明

## 常用命令选项

### save

+ `-k` 或 `--keep-index` 选项： 保存进度后，保持暂存区。
+ `--no-keep-index` 选项： 不保持。
+ `--patch` 选项：交互式选择`stash`与`HEAD`差异，结束后保存进度及暂存区。


### pop | apply


+ `--index`选项：不仅恢复工作区，还恢复暂存区工作进度。

## stash分支

`git stash` 可以将当前工作区内容创建一个新的分支，并检出储藏工作时的所处的提交，重新应用工作，如果成功，将会丢弃储藏。

命令如下:

```shell
git stash branch-name
```


以上便是本篇文章的主要内容。

***

版本修改记录：

+ 2015-07-22 完成本次文章全部内容。 By Octavian
