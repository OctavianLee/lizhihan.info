title: Git Rebase
date: 2015-07-21 22:23:30
tags: [Git, Rebase, 衍合]
categories: Git
---

本文对`git rebase`的使用和实际中的应用做介绍，参考了***Git Pro***的内容。

# Rebase介绍

在日常使用合并的方法有两种：merge(合并)和rebase(衍合)。这里将对`git rebase`命令的使用及注意进行介绍。

## 使用场景
假设一个开发过程，master分支进行过一次合并目前在C4的提交上，而experiment分支是在C2提交上开的分支，目前已经完成准备合并。如图所示：

![rebase使用场景](http://7xjc5f.com1.z0.glb.clouddn.com/img/rebase1-1.png)

此时可以通过`git merge`可以进行使两个分支在共同的C2提交上进行合并。此外，还通过`git rebase`的方法在C4基础上打上C3产生的变化补丁。效果如图所示：

![rebase使用后场景](http://7xjc5f.com1.z0.glb.clouddn.com/img/rebase1-2.png)

## 原理
git rebase，提取两个分支的共同祖先并提取所在分支每次提交的差异并分别保存到临时文件里，切换到需要衍合分支，将所有差异打上补丁。

衍合能产生一个更为整洁的提交历史。


衍合命令如下：

```shell
$ git checkout experiment$ git rebase master
```

衍合后可以切换主分支进行合并。


# 实际案例

## 场景介绍
假设一个开发流程，提交在C2时候，需要对服务器端代码进行特性开发，创建了server分支，进行C3和C4的提交，而在C3的提交的基础上，对客户端进行特性开发，创建了client分支并完成了C8和C9提交。

![rebase实际场景](http://7xjc5f.com1.z0.glb.clouddn.com/img/rebase2-1.png)

## client重演合并
由于进度开发不同，客户端的功能提前完成了，需要提前进行合并，可以通过`git rebase`中的`--onto`选项来进行重演，命令如下：

```shell
$ git rebase --onto master server client
```

之后进行合并，命令如下：

```shell
$ git checkout master
$ git merge client
```

结束后，分支效果如图：

![rebase重演合并](http://7xjc5f.com1.z0.glb.clouddn.com/img/rebase2-2.png)

## server衍合合并

这时，server的开发完成，现在只需要进行衍合操作，便能继续完成剩余合并，命令如下：

```shell
$ git rebase master server
$ git checkout master
$ git merge server
```

结束后，分支效果如图：

![rebase重演合并](http://7xjc5f.com1.z0.glb.clouddn.com/img/rebase2-3.png)

最后进行合并：

```shell
$ git checkout master
$ git merge server
```

## 后续处理

当完成合并之后，删除开发所用分支，完成所有操作：

```shell
$ git branch -d client
$ git branch -d server
```

# 衍合的风险

在公共项目中使用衍合可能会造成提交历史的混乱。

## 场景说明
假设有一个线上公共分支，你作为开发者，将其克隆下来进行开发，如图：

![rebase3-1](http://7xjc5f.com1.z0.glb.clouddn.com/img/rebase3-1.png)

此时其他开发这推送了一次合并到线上，你将其合并到本地开发分支上，此时情况如图：

![rebase3-2](http://7xjc5f.com1.z0.glb.clouddn.com/img/rebase3-2.png)

假设此时推送C6的开发者，采用衍合重新提交，而你的开发分支还是以C6为基础，此时情况如图：

![rebase3-3](http://7xjc5f.com1.z0.glb.clouddn.com/img/rebase3-3.png)


这个时候，你重新拉去合并后，就会产生了相同内容的合并，造成历史记录混乱，如图：

![rebase3-4](http://7xjc5f.com1.z0.glb.clouddn.com/img/rebase3-4.png)

# 高级应用:重写历史

当需要更改最近的历史记录时`git rebase`的`-i`选项进行交互式地重写历史。

重写有六个方式：

1. pick: 选择留下该记录；
2. edit: 修改改次提交内容；
3. squash: 合并改次提交；
4. reword: 修改某次提交的信息；
5. fixup: 类似于squash，但是会放弃这次提交的记录信息；
6. exec: 用shell运行命令。

## 开始重写历史

当需要修改最近三次提交时候，输入:

```shell
$ git rebase -i HEAD~3
```
此时会提供一个提交列表，例如：

```
pick f7f3f6d changed my name a bitpick 310154e updated README formatting and added blame 
pick a5f4a0d added cat-file
```

## 变更历史提交

+ 修改提交

将`pick`修改成`edit`后保存。
之后，撤销提交进行添加，并继续`rebase`过程。命令如下：

```shell
$ git commit --amend
$ git rebase --continue
```

+ 分拆提交

譬如将第二行的记录分拆成两次提交，将`pick`改成`edit`保存后，使用`git reset HEAD^`进行重置，重新提交后，继续`rebase`过程。命令如下：

```shell
$ git reset HEAD^$ git add README$ git commit -m 'updated README formatting' 
$ git add lib/simplegit.rb$ git commit -m 'added blame'$ git rebase --continue
```

+ 删除提交

只需要将需要删除的提交那一行记录进行删除后进行保存。

+ 合并提交

将需要合并的提交，修改`pick`为`squash`后保存，完成合并。


***For Details*** 

参见: http://git-scm.com/docs/git-rebase


以上便是本文全部内容。

***

版本修改记录：

+ 2015-07-21 完成本次文章全部内容。 By Octavian
+ 2015-07-24 补写全部重写的命令方式。 By Octavian