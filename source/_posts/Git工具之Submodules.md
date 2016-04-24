title: Git工具 Submodules
date: 2015-03-18 21:47:12
tags: [Git, Submodules]
categories: Git
---

# 引言

有时候，在使用Git管理仓库时，会需要在一个独立项目中使用另一个独立项目。例如，你的项目中会使用了第三方的库等等，如果单纯的把该库加入进源代码中可能会遇到如下问题：

+ 使用的项目因为需要保证所有的客户都能使用到，很难去定制使用或者去部署；
+ 发布代码到自己的项目时，任何对该部分库的修改都会使这个库在可以更新新版本时变得很难合并；

为了解决这样的问题，Git使用submodules这一个工具。

Submodules能够让一个Git的仓库作为另一个Git仓库的子目录，进行克隆并保持你自己的提交独立。

# 使用Submodules

## 初始化工具
首先确认你需要使用项目仓库，到你的目录下进行初始化操作。
假设你的项目为project，该仓库为Test，Git url为git://github.com/test/test.git 。

***注意：在master分支上进行操作，否则会出现很多不必要的问题。***

### 新项目初始化
在一个没有添加过Test仓库项目直接添加仓库的url，即进入该项目，使用<code>git submodule add</code>命令：

```
cd project
git submodule add git://github.com/test/test.git test
git commit -m 'first commit with submodule test'
git push
```

这样便能将Test仓库添加进project项目中。

### 原有项目初始化
如果本地需要使用线上已添加Test仓库到项目中的project时，直接克隆项目后，Test所在目录为空，需要进行初始化并获取到仓库内容，使用<code>git submodule init</code>命令初始化，并使用<code>git submodule update</code>命令获得仓库中内容，命令如下：

```
git clone git://github.com/test/project.git
cd project
git submodule init
git submodule update
```
或者使用<code>git submodule update --init</code>命令一步完成：

```
git clone git://github.com/test/project.git
cd project
git submodule update --init
```

这样变完成了submodule项目的初始化操作。

## 观察submodule
当使用<code>git submodule add</code>添加了项目，没有提交之前，使用git status会看到两个被commit的文件：

```
# On branch master
# Changes to be committed:
#   (use "git reset HEAD <file>..." to unstage)
#
#      new file:   .gitmodules
#      new file:   test
#
```

### 仓库条目(entry)
test是仓库的条目，当提交之后会显示：

```
create mode 160000 test
```
160000 mode是特定的针对一个目录条目使用的记录。

进入该条目便可以查看改该仓库的内容。

### gitmodules
.gitmodules文件用来存储项目中你使用的submodule信息的配置文件，例如：

```
[submodule "test"]
      path = test
      url = git://github.com/test/test.git
```

## submodule的操作
在使用submodule的项目时，当其他开发者修改了Test仓库代码提交后，使用<code>git pull</code>更新后，这时只是记录了子目录的改变，但子目录下的代码并没有更新，需要使用<code>git submodule update</code>将子目录的代码也进行更新。

这样有时候会遇到这样的问题，当一个开发者修改了submodule的内容，没有进行提交到Git服务器上，之后当开发者提交一个项目本身的提交后，其他项目者更新时候会遭遇如下的错误：

```
git submodule update
fatal: reference isn't a tree: xxxx
Unable to checkout 'xxxx' in submodule path 'test'
```
这时需要他将子模块的修改部分内容提交到线上。

## 删除submodule项目
当不在使用submodule时需要对项目中相关内容进行删除，操作如下：

+ 删除.gitmodulesji及.git/config文件内该仓库的信息；
+ 删除submodule内容（<code>git rm --cached test</code>，<code>rm -rf .git/modules/test</code>，<code>rm -rf test</code>）；
+ 更新项目追踪<code>git add -A ./</code>；
+ 提交所有更新。

# 应用场景
对一个项目分拆，使用submodule进行管理，步骤如下：

+ 新建分拆内容的项目，迁移代码完成提交；
+ 原有项目删除分拆内容，提交合并，切换到master分支上；
+ 进行submodule项目添加，完成项目分拆。

以上便是本篇文章全部内容。


***

版本修改记录：

+ 2015-02-25 完成本次文章全部内容 By Octavian
+ 2015-03-18 完成博客版修改 By Octavian