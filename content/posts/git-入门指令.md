+++ 
draft = true
date = 2021-12-16T13:22:54+08:00
title = "git-入门指令"
description = ""
slug = ""
authors = []
tags = ["git"]
categories = ["git"]
externalLink = ""
series = []
+++


#### 初始化连接 git 相关

```
    git clone xxx(远程仓库名)            ---克隆项目
    git remote add origin xxx           ---本地仓库连接到远程
```

#### 配置账号密码

```
    git config --local user.name xxx
    git config --local user.email 'xxx@xx.com'
```

- git 配置推送时自动保存账号密码: `git config credential.helper store`
- 参考文章: [https://cloud.tencent.com/developer/ask/131910](https://cloud.tencent.com/developer/ask/131910)

#### 单个分支相关

- 创建本地分支: git branch dev/nsm
- 删除本地分支： git branch -D dev/nsm
- 切换到本地分支： git checkout dev/nsm
- 切换上一个分支： git checkout -
- 创建并切换到改分支： git checkout -b xxx

    ...
<!--more-->

- 查看本地分支： git branch
- 查看远程分支： git branch -r
- 查看本地分支和远程分支： git branch -a
- 推送本地分支到远程： git push origin dev/nsm
- 删除远程 git 分支： git push origin -d dev/nsm or git push origin :dev/nsm
- 合并分支: git merge xxx ???
- 更新关联远程分支： git fetch origin
- 刷新远程分支列表： git remote update origin -p
- 基于远程分支新建一个本地分支： git checkout -b xxx origin/xxxx
- 分支操作：
  - 修改分支名： git branch -m 旧分支名 新分支名 ==（这里只是把本地分支重命名， 后续还需要把 远程旧分支删除，然后 push 新分支名==
  - 刷新远程分支： git remote update origin -p

#### 日志相关

- 查看提交日志： git log ==(q: 快速退出)==
- 查看简化日期: git log --oneline
- 查看分支提交树： git log --graph
- 查看提交日志最新 x 次： git log -number
- 查看 log 差异： git log -p -number .
- 查看提交明细： git show
- 查看指定 commit 修改：git show xxx(commit 号)

- 查看本地提交状态： git status
- 查看简化版状态： git status -s
- 查看未加入暂存区(缓冲区)文件的变化： git diff
- 查看已加入暂存区(缓冲区)文件的变化： git diff staged
- 移除文件：
  ```
  git rm 文件名      ( 不在暂存区的文件 )
  git rm -f 强制删除 ( 已在暂存区的文件 )
  ```

#### 查看文件

```
    git blame xxx(文件名)   查看文件提交信息
    cat xxx(文件名)         查看文件
    git reflog              获取全部版本号
```

#### commit 相关

    本地仓库：修改上一次commit信息(已commit)
            git commit --amend -m "备注"

    实际应用中，当完成一次提交之后，可能会发现此次提交有些文件需要修改，当然我们可以在下一次提交中修改此文件
        解决思路：
        git add .
        git commit --amend -m 'new remark'        2次提交合在一起，并且只有共用新的提交备注

#### git cherry-pick

    场景： cherry-pick和它名字一样，挑选一个我们需要的commit进行操作, 它可以用于将其他分支的commit移植到当前分支。
    步骤：

- A 分支有部分新的 commit, 切换 B 分支
- (B 分支) git cherry-pick xxx xxxx xxxxx

#### cherry-pick 出现冲突情况： （总体跟 rebase 操作类似）

1、git status 查看冲突文件 => 手动解决冲突 => git add .  
2、git cherry-pick --continue  
3、放弃 cherry-pick => git cherry-pick --abort

#### 提交代码代码相关

- 拉取远程代码： git fetch origin xxxx
- git merge origin/xxx
- git add . (将文件加入到缓冲区,加入缓冲区后，发 p 现文件名变成绿色) --暂存区
- git reset HEAD . --撤离暂存区
- git commit -m "这里是备注". --提交更新
- git push -u origin master(分支名) 推送到远程分支(第一次提交) ---推送到远程
- git push origin master(分支名) 推送到远程分支

#### 文件回退

```
    撤回所有文件更改： git checkout .
    修改文件未 git add., 那么直接 git checkout 文件名
    加入到缓存区后 git add .  =>    撤销暂存区更新： git reset HEAD fileName    => 然后git checkout

    回退本地仓库：
        回滚有三种方式
        1、git reset --hard xxx         本地仓库的暂存区和工作区都将回滚到指定commit号(一般不建议使用)
        2、git reset --soft xxx         本地仓库回滚到指定commit号，暂存区和工作区文件状态不会有任何变化
        3、git reset --mixed xxx        本地仓库回滚到指定commit号，会重置暂存区，同时工作区的文件会处理未被跟踪状态
        强制推送远程仓库
            1、git push -f origin xx(远程仓库名)        强制push到远程仓库
            2、 git reflog                             获取全部版本号

    回退远程仓库：
        git reset --hard origin/master   强制远程master 覆盖本地master
```

#### 远程更新本地仓库

```
    git checkout xxx
    git fetch origin xxx   更新远程仓库同步到本地（不会merge）
    git merge origin/xxx   本地仓库merge远程仓库，本地commit 同步
```

#### 删除文件

```
    commit后恢复文件 => 删除文件 => 恢复文件 || add.
    执行步骤： git checkout -- 文件名
    查看工作区要被删除的文件 git clean -n
    删除工作区文件： git clean -f
```

#### 远程相关

```
    git remote          查看远程信息
    git remote -v       查看远程库详细信息
    github重命名远程仓库:
        1、github上 setting => 修改库名称
        2、本地git项目上有以下操作
            @ git remote rm origin
            @ git remote add origin git@github.com:nieshiming/xxx.git
```

#### 概念相关

**工作区**： 就是电脑上看到的目录，日常所建的目录都是属于工作区的范畴  
**版本库**：工作区有一个隐藏目录.git, 这个不属于工作区，这是版本库，版本库存在了很多东西，其他最重要的  
 就是暂存区，还有 git 为我们自动创建的 master 分支，以及只想 master 分支的 head

**head**: 指向工作所在的分支  
==git 提交文件到版本库有 2 步==

- 1、使用 git add . 把文件添加进去，实际就是把文件添加到暂存区
- 2、使用 git commit 把暂存区的内容提交到当前本地仓库的分支上去

#### rebase 相关

```
rebase 实例： {
    git checkout master
    git fetch origin
    git merge origin/master

    git checkout development
    git fetch origin
    git merge origin/development

    git rebase origin/master
    git checkout master
    git merge development
}
```

##### git rebase 原理：

- 基于远程 origin/nsm 拉出来一个 nsm 本地分支：
- 对应文章： [https://www.cnblogs.com/kevingrace/p/5896706.html](https://www.cnblogs.com/kevingrace/p/5896706.html)

```
条件：
    1、origin/nsm 有提交
    2、nsm本地分支有提交
生成条件：
    git fetch origin nsm
    git rebase  origin/nie  以远程复位基底
    git push origin nsm
```

##### git rebase 基于同一分支有修改有冲突

```
条件：
   1、origin/nsm 有提交
   2、nsm本地分支有提交
生成条件：
   git fetch origin nsm
   git rebase origin/nsm
   /****解决冲突****/
   查看冲突文件个数： git status --uno
   /****解决冲突****/
   git add .
   git rebase --continue
   git push origin nsm

不同分支：
   基于 relase1.3 开分支nsm分支
   git rabase origin/release1.0
   nsm 分支提交
   checkout release1.3
   git merge nsm
   git fetch origin/relase1.3
   git rebase origin/releasa1.3
   git push origin release1.3
```

##### fast forward

描述： 分支间快速推进；
形成条件： A 分支没有 commit , B 分支是基于 A 分支切出来的分支， 并有 commit  
执行步骤
1、执行 **A merge B** ; => A 分支快速合并 B 的 commit， ==不会产生新的 commit==
2、三方合并：A 分支上切分支 B， A B 各有新的 commit, A merge B 会形成新的 commit 来链接 2 个 commit

##### git rebase 原理

条件：A 分支上切分支 B， A B 各有新的 commit

- A rebase B, A 将除去共有祖先上的 commit 全部以 B 为基底嫁接到分支 B 上去  
   _这样 A 分全部 B 分支 commit 为基点，嫁接自身新增的 commit_
- 回到 B 分支，直接 merge A 分支，执行快速推荐
- [参考文章](https://www.jianshu.com/p/ca76937b174f)

```
Rebase - 使用 Interactive 模式來精簡 commit 紀錄
1、基于master分支开了一个dev分支, 同时master 、 dev 分支都有commit
2、切换dev分支 git rebase -i master
3、确认无误后, 输入: wq 退出
4、切换master分支, 执行git merge dev
```

#### tag 标签相关

```
tag相关：
    查看tag: git tag
    打tag:
        1、切换目标分支： git tag xxx
        2、针对同一个commit号，可以打多个tag
    删除本地tag: git tag -d xxx
    针对某个commit号打tag  git tag nie-t-4 cdd9b49
    查看某次tag详细信息: git show xxx
    推送tag到远程： git push origin xxx
    推送全部tag到远程:  git push origin --tags
    删除远程tag：
        1、删除本地tag: git tag -d xxx
        2、更新至远程： git push origin :refs/tags/xxx
```

#### stash (存储相关)

描述： git stash 命令的作用就是将目前还不想提交的但是已经修改的内容进行保存至堆栈中，后续可以在某个分支上恢复出堆栈中的内容(不是当前分支上的 stash 也可以应用)

```
  git stash 保存当前缓存
  git stash list 查看缓存列表
  git stash apply 应用最近的一次缓存
  git stash apply stash@{x} 应用指定的缓存

  git stash show -p 查看缓存更改明细
  git stash pop 应用之前最近一次保存的缓存，并且在缓存栈中删掉
                如果应用并删除指定缓存可以： git stash pop stash@{x}
  git stash drop stash@{x} 删除指定缓存
  git stash clear 清空缓存栈
```

#### git 配置别名

```
1、安装cmder
2、vscode配置cmder终端
   - https://github.com/Microsoft/vscode/issues/12006
3、cmder配置git 别名
   - https://github.com/cmderdev/cmder/issues/421
   - 进入到  E:\ruanjian\cmder\config\user_aliases.cmd配置别名
             https://www.jianshu.com/p/979db1a96f6d
4、重启vscode
```


#### git 教学文章

- [https://gitbook.tw/](https://gitbook.tw/)
- [https://backlog.com/git-tutorial/tw/stepup/stepup2_8.html](https://backlog.com/git-tutorial/tw/stepup/stepup2_8.html)
- [readMe添加emoji](https://www.webfx.com/tools/emoji-cheat-sheet/)

#### git 提交规范

1、[https://segmentfault.com/a/1190000009048911](https://segmentfault.com/a/1190000009048911)  
2、[https://github.com/angular/angular/blob/master/CONTRIBUTING.md](https://github.com/angular/angular/blob/master/CONTRIBUTING.md)  
3、[https://github.com/angular/angular-cli/commits/master](https://github.com/angular/angular-cli/commits/master)

#### git 分支管理规范

1、参考文章： [https://cloud.tencent.com/developer/article/1110910](https://cloud.tencent.com/developer/article/1110910)

```
    <type>(作用域): 描述
    注意：作用域可精确到model，interface, service，view ，*以及具体到那个文件等等
```