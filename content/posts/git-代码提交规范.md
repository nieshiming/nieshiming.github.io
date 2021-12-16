+++ 
draft = true
date = 2019-03-16
title = "git-代码提交规范"
description = ""
slug = ""
authors = []
tags = ["git"]
categories = ["git"]
externalLink = ""
series = []
+++

#### 代码提交规范
因为最近项目比较多，而且经常穿插各种很紧急的小需求，导致版本管理比较困难，所以现在定制一下代码的提交规范

#### 具体步骤
- 基于master分支，创建出自己的开发分支，分支命名可以参考 `feature-功能描述` 或者 `bug-bug描述` 或者 `develop-姓名` 这样的模式。
- 在自己的开发分支上修改代码，创建Commit。
- 在开发完成之后，将自己本地的开发分支推送到远端，这时候需要确定自己的开发分支需要合并进哪个版本分支。
- 在明确了版本分支之后，在GitLab上创建Merge Request。创建Merge Request的步骤如下：
    ...
    <!--more-->
    - 打开项目的git地址，在左侧找到**Merge Request**的链接，点击进入Merge Request，然后点击右上角的**New Merge Request** 按钮 ![image](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dc964aa81add470cb231d9bcd00a2646~tplv-k3u1fbpfcp-watermark.image)
    - 选择你的分支和开发分支。![image](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ee20783a344243c7ab1e88f28f749b6f~tplv-k3u1fbpfcp-watermark.image)
    - 填写信息，指定人员Review你的代码 ![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ecf467fe0925491595895c8645ee04a9~tplv-k3u1fbpfcp-watermark.image)
    - 提交Merge Request,GitLab会根据你的代码情况展示不同的界面。
        ![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/393607a0faf8415092a0a4dfbcc36194~tplv-k3u1fbpfcp-watermark.image)
    
---

#### 声明几个需要注意的操作
[angularjs提交规范](http://www.ruanyifeng.com/blog/2016/01/commit_message_change_log.html)

- 每一次commit,都有尽可能的把commit的内容描述清楚，禁止用类似~~修复bug~~、~~修改代码~~这样模糊不清的描述作为commit msg 提交。
```ssh
    // Bad  应该描述出修复了哪个模块的什么bug
    commit eaee23a5c28f02c28059fa6c91df898ca41730a8
    Author: xxx <xxx@lechebang.com>
    Date:   Thu Dec 6 17:02:32 2018 +0800
    desc: "adjust"

    // Good
    commit f1c8c64cca0d4d627c5f445f239ead4db13c4fd1
    Author: xxx <xxx@lechebang.com>
    Date:   Tue Dec 18 17:04:39 2018 +0800
    descL "chore(xxx): 修复了xxx,调整xxx，解决了"
```
- 禁止在版本分支上直接提交commit。
-  为了保证公共分支(master、版本分支)的提交记录清晰，禁止执行`git merge`指令去将公共分支的代码合并进开发分支。如果需要同步公共分支的代码，可以通过`git rebase 公共分支`去同步。
> 比如我们可能1.1.0和1.2.0的版本同时在开发，1.1.0的版本分支会在1.2.0之前合并进master分支，所以在1.2.0提测之前，执行`git rebase master`去同步master的代码到1.2.0
> 在Merge Request 出现无法合并的情况，一般都是因为目标分支和你的开发分支有代码冲突，所以在你本地的开发分支上执行`git rebase 目标分支`去解决冲突，冲突解决完成之后重新把你的分支推送到远端即可。(这个步骤不需要重新发起Merge Request)

- 自己的开发分支在rebase了其他分支的代码之后，推送远端需要增加`--force` 选项去强制更新远端的开发分支。但是如果你要强制更新一个远端的公共分支时，请三思...如果是自己的分支可以随便玩，公共分支的所有操作都要慎重。
- 现在我们一直没有代码review的过程，是因为这个过程太难推进了，工作量巨大，但是之后会整理出一些基本的代码规范，希望大家可以按照代码规范去进行开发。