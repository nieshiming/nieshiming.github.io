+++ 
draft = true
date = 2021-12-16T13:24:46+08:00
title = "git-pull-和git-fetch区别"
description = ""
slug = ""
authors = []
tags = ["git"]
categories = ["git"]
externalLink = ""
series = []
+++


#### 描述

> git fetch并没有更新远程代码到本地仓库，只是拉去了远程的commit数据，将远程仓库的commitid 更新为lastet

##### 工作区
简而言之就是工作的区域，对于git而言，就是当前工作的目录，工作的内容会被提交到暂存区以及版本库中，同时也包含自己的修改的内容

##### 暂存区
使用git add 命令把我们工作区修改的内容添加到暂存区中

<!--more-->

##### 本地仓库
使用git commit命令后，会把暂存区的内容提交到本地仓库中，工作区有个.git目录，这个目录不属于工作区，里面是关于仓库的相关信息， 暂存区的内容也会在里面。记录相关commit节点，
可以使用git merge，或者rebase命令把远程仓库副本合并到本地仓库

##### 远程版本库(远程代码库)
与本地仓库概念一致，只不过存储于远程，可用于远程多人协作，可以通过git pull 、 git push来下载或推送代码，实现本地仓库与远程仓库的交互

##### 远程仓库副本
可以理解为存储于本地的远程仓库缓存，记录远程仓库的相关信息，主要是commit节点， git fetch只是同步远程仓库数据至本地的远程仓库副本， 可以通过git pull拉取远程仓库代码只本地仓库
**git pull根据配置不同**分为以下2种形式
```ssh

    git pull === git fetch + git rebase branch

    git pull === git fetch + git merge branch

```

#### 实践

> 查看指定仓库的 .git文件夹。 .git文件夹中refs里面有3个文件夹： headers、remotes、tags。其中heads记录的是本地仓库最新commit, remotes记录的是远程的commitId，使用git fetch可以把remotes中commit更新至最新


![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/059a4105b46e428d9e7c83192f796a03~tplv-k3u1fbpfcp-watermark.image)

- 使用git fetch命令同步远程commit至remotes

- 在refs中 heads查看本地指定分支<master>commit： eg: 66d961889ed447a37ce612899c954ec4f1363289
    ![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/58db50750cad4f3fa50dc8418dd29a62~tplv-k3u1fbpfcp-watermark.image)

- 在refs中，remotes查看指定分支<master>远程仓库commit数据: 
    ![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/408ba665d0354fce88436bf70a887cee~tplv-k3u1fbpfcp-watermark.image)

- 执行git merge操作
    ![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3f6402ce5c3e4920b63dab3a15f49088~tplv-k3u1fbpfcp-watermark.image)

    ![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/816f4d6d8496435e854e429ea5621aee~tplv-k3u1fbpfcp-watermark.image)


