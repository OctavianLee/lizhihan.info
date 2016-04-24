title: Git Flow流程介绍及使用
date: 2015-04-20 19:23:03
tags: [Git,Git Flow]
categories: Git
---

在日常开发里，我们常使用Git来做代码的版本控制管理，通过Branch的创建合并修改进行分工合作，但是光是有这样的方式开发依旧不能是一个有条理的方式，这个时候我们需要Git Flow来协助我们更好的进行管理开发。

# Git Flow的相关概念

## 什么是Git Flow

Git Flow是基于GIT的开发流程规范，能够通过创建有分别的分支进行管理开发，有效的进行开发合作的工具。
关于工具的介绍详见：https://github.com/nvie/gitflow

## 分支模型（Branching Model）

为了有效的管理分支，Vincent Driessen在他的博客里提出了一个分支模型的概念，并成为Git Flow工具的实践理论参考。
关于博客原文详见：http://nvie.com/posts/a-successful-git-branching-model

### 分支概述
这里结合下图，对博文内容中的分支做一个概述性的说明：
![Git Model](http://myto-team.qiniudn.com/image/f/3e/cc3cf5b3902dbb1ff2356fe65e67e.png)
如图所示,一个项目需要被分成master、develop、feature、release、hotfix五个分支。
根据类别可以分为主要分支和辅助分支两个类别：

+ 主要分支：master、develop
+ 辅助分支：feature、release、hotfix

### 主要分支
![main branches](http://myto-team.qiniudn.com/image/5/54/f4577fd87859c7d7ae840f05c3416.png)
#### 主分支（master）
master是git创建项目后默认在orign中的，他的头(HEAD)永远是produciton-ready的状态。也是当每个项目发布的最终版本的分支。
#### 开发分支（develop）
develop分支是所有代码头是记录下一个发布版本的最新更新的分支。
当develop分支的代码已经可以被发布的时候，他将会被合并到主分支上并打上发布标签。

### 辅助分支
辅助分支是帮助团队成员进行协同开发使用的，它可以比较容易追踪特性，准备产品发布且快速修复产品中的问题，每个辅助分支有着不同的作用和界限。
#### 特性分支（feature branches）
特性分支是用来开发下一个版本的新特性使用的分支，他在开发完成后合并到开发分支中。也就是说一般开发新功能都是要创建这个分支。

![feature branches](http://myto-team.qiniudn.com/image/d/a6/2ac4baa44ce986ec8a8e89d487317.png)

一般来说特性分支遵从下面规范：

+ 来源如无特别情况是develop；
+ 必须合并到develop上；
+ 命名规则除了master、develop、release-*、hotfix-* 以外的命名。

#### 发布分支（release branches）
发布分支是为产品发布做准备的分支,他可以修复小的bug且准备发布元信息（版本号码、创建日期等）。也就是发布到master前的最后一步准备。

一般来说发布分支遵从下面规范：

+ 来源如无特别情况是develop；
+ 必须合并到develop和master上；
+ 命名规则为release-*的形式。

#### 修复分支（hotfix branches）

修复分支主要是突发情况下对发布产品进行版本BUG修复的分支。如果遇到重大BUG，需要立即创建分支进行修复合并。

![hotfix brances](http://myto-team.qiniudn.com/image/2/99/a5c36e850fbda603ded8ad1f02169.png)

一般来说修复分支遵从下面规范：

+ 来源如无特别情况是master；
+ 必须合并到develop和master上；
+ 命名规则为hotfix-*的形式。

## 团队流程

在介绍完分支模型后，我们团队结合我们的开发情况并基于Git Flow流程的开发应当是这样的：

+ 团队负责人，当新建一个Git项目后，master自动生成；
+ 团队负责人创建生成develop分支；
+ 团队成员当需要新功能时，创建生成该功能feature分支并开发；
+ 团队成员每日任务完成并推送到线上对应分支上去；
+ 当开发完成无误，团队负责人确认后将分支合并入develop中并更新线上develop;
+ 在develop阶段性完成后，团队负责人创建发布分支，并写上发布信息，后并入master中；
+ 如果发布分支遇到小的bug修复后，由团队负责人并入develop和master中；
+ 如果遇到产品BUG，团队成员创建修复分支，成功修复后由团队负责人合并入develop和master中。

以上便是Git Flow流程的介绍。

# Git Flow工具的使用

## Git Flow安装

安装方式根据系统不同可以是Mac 、Linux、Windows 等方式，详细见：
https://github.com/nvie/gitflow/wiki/Installation

## Git Flow初始化

在安装好Git Flow到你需要进行Git Flow的项目中去进行操作，譬如我们针对icosta项目开始Git Flow 流程的开发：

```
$ cd icosta
$ git flow init -d
```

这样便按默认方式对项目进行了初始化操作，但是目前的会有这样的情况，你在远程有了develop分支了，可是你init后也创建了分支，这样需要将develop的合并：

```
$ git merge origin/develop
```
这样你就可以正常的使用工具了，下面谈谈特定分支下的git flow的操作吧！

## Git Flow分支操作
Git Flow分支的操作方式，在官方wiki中有所介绍：
https://github.com/nvie/gitflowiki/Command-Line-Arguments
本部分内容只是介绍一些最常见的功能。

### 特性分支开发
这边我们通过特性分支的使用讲解git flow具体场景下的如何操作，其他的各个分支的使用类似。

#### 开始分支

首先我们需要开设一个关于发帖功能在icosta项目中，我们使用命令如下：

```
$ git flow feature start post
```
这样我们会基于develop分支创建一个在featrue/下的post分支。

#### 追踪文件
当你开发完成了一部分功能后，譬如完成发帖功能，下面你需要把做的内容提交更新到post分支下，在此之前你需要把所有涉及到的文件进行追踪。
首先需要你可以通过：

```
$ git status
```
来查看到所有你此次更新涉及到的文件，其中N为新增，M为修改，D为删除。
接着添加新增和修改的文件：

```
$ git add <涉及到的文件>
```
再移除那些删除掉的文件:

```
$ git rm <涉及到的文件>
```
当然如果文件过多，也可以一次新完成所有追溯包含删除文件，譬如我们需要完成对icosta文件夹下所有文件进行追溯：

```
$ git add --all icosta/
```
这样便完成所有文件的追踪。

#### 确认提交

下面对此次进行确认命令如下：

```
$ git commit -m "the implement of creating a post"
```
接着将代码从本地提交到post分支上：

```
$ git push origin feature/post
```
这样我们便完成了确认提交。

#### 参与开发
当其他人参与了这个功能的开发，为了让他能够参与到项目中，他负责其他的删除、修改帖子的功能，此时他需要先从服务器上下载代码：

```
$ git flow feature pull origin post
```
如果这时，你因为计划变动，需要且完成修改帖子功能，更新到服务器上，新参与者需要同步更新代码，这时候他需要：

```
$ git flow feature rebase post
```
如果有冲突，手工修改，遵照这样的方式可以协同完成一个分支的工作。

#### 完成分支

当post功能完成后，且确认通过测试，这时团队负责人需要完成这个特性进行提交，命令如下：

```
$ git flow finish post
```
这样这个分支会将历史修改合并起来提交到develop,再没有冲突后，填写更新日志，完成提交。

如果通过确认，该分支会被删除，此时你需要更新一下本地的develop分支为下一次开发做准备：

```
$ git pull origin develop
```
### 其他

#### 版本号tag

当完成发布版本之后，需要用：

```
$ git push --tags 
```
将该发布打上版本号。

#### 关于support

Git flow还定义了一种support分支，这个作为辅助性的分支，目前我们团队暂时不使用，这里不做过多介绍。

####publish、track使用

Git flow的方法中也定义了publish和track的命令，但是基于我们的项目开发流程，这里也是不需要使用的。

以上便是本Git Flow流程介绍及使用的全部内容。


***

版本修改记录：

+  2014-11-11 完成本次文章全部内容 By Octavian
+  2014-11-13 基于使用内容，调整git flow命令使用，增添关于publish和track未使用的说明 By Octavian
+  2015-04-21 发布博客版本 By Octavian
+  2015-05-27 格式修改 By Octavian