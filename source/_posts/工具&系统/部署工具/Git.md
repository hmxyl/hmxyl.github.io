---
title: Git
date: 2022-10-31 21:32:25
tags: 
  - Git
categories: 
  - [工具&系统, 部署工具, Maven]
description: Git日常使用记录
---



# 场景命令

## 一、切换分支临时处理之后恢复

经常有这样的事情发生，当你正在进行项目中某一部分的工作，里面的东西处于一个比较杂乱的状态，而你想转到其他分支上进行一些工作。问题是，你不想提交进行了一半的工作。解决这个问题的办法就是`git stash`命令。

现在你在分值aaa，已经做了一些修改，现在想要分支bbb做一些事情，但又不想提交aaa上的一些修改：

```sh
# 保存分支aaa的工作状态
git stash

# 切换到分支bbb
git checkout bbb

# 在分支bbb上做一些操作后，返回分支aaa
git checkout aaa

# 恢复之前的工作状态
git stash apply 或者 git stash pop
```



## 二、处理 Git 忘记切分支修改了代码的情况

有时候没注意分支，直接在 master 上做开发了，假设你现在在 master 分支上已经修改了文件：

```sh
# 把当前未提交到本地（和服务器）的代码推入到 Git 的栈中：
$ git stash

# 查看效果：
$ git status 

# 切换分支：
$ git branch dev 

# 还原代码：
$ git stash apply
```



## 三、本地新建分支后，同步到远程不存在的分支

```sh
$ git push local-branch-name:remote-branch-name
```

## 四、撤销某次commit

```sh
# 先找到commit id
$ git log

# 撤销
$ git reset --hard commit_id
```



## 五、重命名分支

```sh
1、本地分支重命名
 git branch -m oldName  newName
 
2、将重命名后的分支推送到远程
git push origin newName
```



## 六、分支覆盖

```sh
git checkout pre-release
git reset --hard origin/develop
git push -f
```

## 七、查看日志

```sh
git log --graph --pretty=oneline --abbrev-commit
```



## 八、合并多个commit为另外的总commit

```
参考：https://backlog.com/git-tutorial/cn/stepup/stepup7_5.html
```

当前分支：分支1
待合并分支：分支2

```sh
$ git checkout -b 分支2 origin/分支2
$ git checkout 分支1
$ git merge --squash 分支2
$ git commit -m 'PDF fix1'
$ git push origin
```



实际应用

```sh
git fetch origin -p
git pull
git branch -m  feature-11111 feature-11111-1
git push origin --delete feature-11111
git checkout -b dev origin/dev
git branch -m dev feature-11111
git checkout feature-11111
git push origin feature-11111
git branch -m feature-11111 dd
git checkout -b feature-11111 origin/feature-11111
git branch -D dd

git merge --squash feature-11111-1

git commit -m 简介
git push origin feature-11111
git branch -D feature-11111-1
git branch -m feature-11111 dd
git checkout -b feature-11111 origin/feature-11111
git branch -D dd
```



## 九、解决git文件名大小写无法修改的问题

```
git默认配置为忽略大小写，因此无法正确检测大小写的更改

运行：
git config core.ignorecase false

关闭git忽略大小写配置，即可检测到大小写名称更改
```



# 常用命令

## 一、流程图示

![img](D:\0_Notes\Hexo\hmxyl\source\_images\f96b3444-14a7-4793-a4a5-a0bd3d52b576.png)

Git中几个专用名词的译名如下：

```
Workspace：工作区
Index / Stage：暂存区
Repository：仓库区（或本地仓库）
Remote：远程仓库
```

## 二、新建代码库

```sh
# 在当前目录新建一个Git代码库
$ git init

# 新建一个目录，将其初始化为Git代码库
$ git init [project-name]

# 下载一个项目和它的整个代码历史
$ git clone [url]
```

## 三、配置

Git的设置文件为`.gitconfig`，Git的配置分为两种：

- 全局配置：在用户主目录下
- 在项目目录下

注意：在当前项目下面查看的配置（`git config --list`）是全局配置 + 当前项目的配置，使用的时候会优先使用当前项目的配置；

一般公司项目都是在GitLab上的，所以可以在项目根目录进行单独配置，不用全局设置，以免影响其他远程仓库如GitHub的使用。

```sh
# 显示当前的Git配置
$ git config --list

# 编辑Git配置文件
$ git config -e [--global]

# 设置提交代码时的用户信息
$ git config [--global] user.name "[name]"
$ git config [--global] user.email "[email address]"
```



## 四、增加/删除文件

```sh
# 添加指定文件到暂存区
$ git add [file1] [file2] ...

# 添加指定目录到暂存区，包括子目录
$ git add [dir]

# 添加当前目录的所有文件到暂存区
$ git add .

# 添加每个变化前，都会要求确认
# 对于同一个文件的多处变化，可以实现分次提交
$ git add -p

# 删除工作区文件，并且将这次删除放入暂存区
$ git rm [file1] [file2] ...

# 停止追踪指定文件，但该文件会保留在工作区
$ git rm --cached [file]

# 改名文件，并且将这个改名放入暂存区
$ git mv [file-original] [file-renamed]
```



## 五、代码提交

```sh
# 提交暂存区到仓库区
$ git commit -m [message]

# 提交暂存区的指定文件到仓库区
$ git commit [file1] [file2] ... -m [message]

# 提交工作区自上次commit之后的变化，直接到仓库区
$ git commit -a

# 提交时显示所有diff信息
$ git commit -v

# 使用一次新的commit，替代上一次提交
# 如果代码没有任何新变化，则用来改写上一次commit的提交信息
$ git commit --amend -m [message]

# 重做上一次commit，并包括指定文件的新变化
$ git commit --amend [file1] [file2] ...
```

## 六、分支

```sh
# 列出所有本地分支
$ git branch

# 列出所有远程分支
$ git branch -r

# 列出所有本地分支和远程分支
$ git branch -a

# 新建一个分支，但依然停留在当前分支
$ git branch [branch-name]

# 新建一个分支，并切换到该分支
$ git checkout -b [branch]

# 新建一个分支，指向指定commit
$ git branch [branch] [commit]

# 新建一个分支，与指定的远程分支建立追踪关系
$ git branch --track [branch] [remote-branch]

# 切换到指定分支，并更新工作区
$ git checkout [branch-name]

# 切换到上一个分支
$ git checkout -

# 建立追踪关系，在现有分支与指定的远程分支之间
$ git branch --set-upstream [branch] [remote-branch]

# 合并指定分支到当前分支
$ git merge [branch]

# 选择一个commit，合并进当前分支
$ git cherry-pick [commit]

# 删除分支
$ git branch -d [branch-name]

# 删除远程分支
$ git push origin --delete [branch-name]
$ git branch -dr [remote/branch]
```



## 七、标签

```sh
# 列出所有tag
$ git tag

# 新建一个tag在当前commit
$ git tag [tag]

# 新建一个tag在指定commit
$ git tag [tag] [commit]

# 删除本地tag
$ git tag -d [tag]

# 删除远程tag
$ git push origin :refs/tags/[tagName]

# 查看tag信息
$ git show [tag]

# 提交指定tag
$ git push [remote] [tag]

# 提交所有tag
$ git push [remote] --tags

# 新建一个分支，指向某个tag
$ git checkout -b [branch] [tag]
```

## 八、查看信息

```sh
# 显示有变更的文件
$ git status

# 显示当前分支的版本历史
$ git log

# 显示commit历史，以及每次commit发生变更的文件
$ git log --stat

# 搜索提交历史，根据关键词
$ git log -S [keyword]

# 显示某个commit之后的所有变动，每个commit占据一行
$ git log [tag] HEAD --pretty=format:%s

# 显示某个commit之后的所有变动，其"提交说明"必须符合搜索条件
$ git log [tag] HEAD --grep feature

# 显示某个文件的版本历史，包括文件改名
$ git log --follow [file]
$ git whatchanged [file]

# 显示指定文件相关的每一次diff
$ git log -p [file]

# 显示过去5次提交
$ git log -5 --pretty --oneline

# 显示所有提交过的用户，按提交次数排序
$ git shortlog -sn

# 显示指定文件是什么人在什么时间修改过
$ git blame [file]

# 显示暂存区和工作区的差异
$ git diff

# 显示暂存区和上一个commit的差异
$ git diff --cached [file]

# 显示工作区与当前分支最新commit之间的差异
$ git diff HEAD

# 显示两次提交之间的差异
$ git diff [first-branch]...[second-branch]

# 显示今天你写了多少行代码
$ git diff --shortstat "@{0 day ago}"

# 显示某次提交的元数据和内容变化
$ git show [commit]

# 显示某次提交发生变化的文件
$ git show --name-only [commit]

# 显示某次提交时，某个文件的内容
$ git show [commit]:[filename]

# 显示当前分支的最近几次提交
$ git reflog
```

## 九、远程同步

```sh
# 下载远程仓库的所有变动
$ git fetch [remote]

# 显示所有远程仓库
$ git remote -v

# 显示某个远程仓库的信息
$ git remote show [remote]

# 增加一个新的远程仓库，并命名
$ git remote add [shortname] [url]

# 取回远程仓库的变化，并与本地分支合并
$ git pull [remote] [branch]

# 上传本地指定分支到远程仓库
$ git push [remote] [branch]

# 强行推送当前分支到远程仓库，即使有冲突
$ git push [remote] --force

# 推送所有分支到远程仓库
$ git push [remote] --all
```

## 十、撤销

```sh
# 恢复暂存区的指定文件到工作区
$ git checkout [file]

# 恢复某个commit的指定文件到暂存区和工作区
$ git checkout [commit] [file]

# 恢复暂存区的所有文件到工作区
$ git checkout .

# 撤销git add
# 如果是撤销所有的已经add的文件:  
git reset HEAD .
# 如果是撤销某个文件或文件夹（filename：文件名或者文件夹名）
git reset HEAD -filename

# 重置暂存区的指定文件，与上一次commit保持一致，但工作区不变
$ git reset [file]

# 重置暂存区与工作区，与上一次commit保持一致
$ git reset --hard

# 重置当前分支的指针为指定commit，同时重置暂存区，但工作区不变
$ git reset [commit]

# 重置当前分支的HEAD为指定commit，同时重置暂存区和工作区，与指定commit一致
$ git reset --hard [commit]

# 重置当前HEAD为指定commit，但保持暂存区和工作区不变
$ git reset --keep [commit]

# 新建一个commit，用来撤销指定commit
# 后者的所有变化都将被前者抵消，并且应用到当前分支
$ git revert [commit]

# 暂时将未提交的变化移除，稍后再移入
$ git stash
$ git stash pop
```

## 十一、其他

```sh
# 生成一个可供发布的压缩包
$ git archive
```



# 给本地分支添加备注信息

## 安装全局插件

```bat
npm i -g git-br
```

## 查看备注信息（安装插件）

```sh
git-br
```

## 查看信息（未安装插件）

```sh
git config branch.feature_20150713_hd-123.description
```

## 给分支添加备注

```sh
git config branch.feature_20150713_hd-123.description 海南放款
```

