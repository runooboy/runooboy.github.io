---
title: git笔记
date: 2019-10-14 11:00:07
category:
- tools
tags:
- git
---

### 前言

关于git的一些介绍以及大部分用法，可以参考[官方文档](https://git-scm.com/book/zh/v2),本文只记录一些常用操作和一些比较难的操作。

<!-- more -->

### git fetch和git pull对比

先用一张图来理一下git fetch和git pull的概念：

![](git.jpg)

`git fetch`是将远程主机的最新内容拉到本地，用户在检查了以后决定是否合并到工作本机分支中。

`git fetch origin`查找 “origin” 是哪一个服务器,从中抓取本地没有的数据，

`git pull`则是将远程主机的最新内容拉下来后直接合并，即：`git pull` = `git fetch` + `git merge`，这样可能会产生冲突，需要手动解决。

**git fetch用法**

git fetch 命令：
```bash
git fetch <远程主机名> 
//这个命令将某个远程主机的更新全部取回本地,
并且更新本地数据库，移动 origin/所有分支 指针指向新的、更新后的位置。

git fetch (后面不加参数)
// 这个命令与上面的命令相似，远程主机用的默认远程主机
```

如果只想取回特定分支的更新，可以指定分支名：
```bash
git fetch <远程主机名> <分支名> //注意之间有空格
```

最常见的命令如取回origin 主机的master 分支：
```bash
git fetch origin master
```

取回更新后，会返回一个FETCH_HEAD ，指的是某个branch在服务器上的最新状态，我们可以在本地通过它查看刚取回的更新信息：
```bash
git log -p FETCH_HEAD
```

**git pull 用法**

前面提到，git pull 的过程可以理解为：
```bash
git fetch origin master //从远程主机的master分支拉取最新内容 
git merge FETCH_HEAD    //将拉取下来的最新内容合并到当前所在的分支中
```
即将远程主机的某个分支的更新取回，并与本地指定的分支合并，完整格式可表示为：
```bash
git pull <远程主机名> <远程分支名>:<本地分支名>
```
如果远程分支是与当前分支合并，则冒号后面的部分可以省略：
```bash
git pull origin dev
```

### git push

git push的一般形式为 `git push <远程主机名> <本地分支名>  <远程分支名>` ，例如 git push origin master，即是将本地的master分支推送到远程主机origin上的对应master分支， origin 是远程主机名,第一个master是本地分支名，第二个master是远程分支名。

如果远程分支被省略，如下则表示将本地分支推送到与之存在追踪关系的远程分支（通常两者同名），如果该远程分支不存在，则会被新建
```bash
git push origin master
```

如果省略本地分支名，则表示删除指定的远程分支，因为这等同于推送一个空的本地分支到远程分支，等同于 `git push origin --delete master`
```bash
git push origin:master
```

如果当前分支与远程分支存在追踪关系，则本地分支和远程分支都可以省略，将当前分支推送到origin主机的对应分支 
```bash
git push origin
```

如果当前分支只有一个远程分支，那么主机名都可以省略，形如 `git push`，可以使用`git branch -r `，查看远程的分支名
```bash
git push
```

**git push 的其他命令**
1. `git push -u origin master` 如果当前分支与多个主机存在追踪关系，则可以使用 -u 参数指定一个默认主机，这样后面就可以不加任何参数使用git push，不带任何参数的git push，默认只推送当前分支，这叫做simple方式，还有一种matching方式，会推送所有有对应的远程分支的本地分支， Git 2.0之前默认使用matching，现在改为simple方式,如果想更改设置，可以使用git config命令。git config --global push.default matching OR git config --global push.default simple；可以使用git config -l 查看配置
2. `git push --all origin` 当遇到这种情况就是不管是否存在对应的远程分支，将本地的所有分支都推送到远程主机，这时需要 -all 选项

3. `git push --force origin` git push的时候需要本地先git pull更新到跟服务器版本一致，如果本地版本库比远程服务器上的低，那么一般会提示你git pull更新，如果一定要提交，那么可以使用这个命令。

4. `git push origin --tags` //git push 的时候不会推送分支，如果一定要推送标签的话那么可以使用这个命令

### 常用分支操作
1. git 本地分支操作
	```bash
	git branch //查看本地所有分支 
	
	git branch -r //查看远程所有分支
	
	git branch -a //查看本地和远程的所有分支
	
	git branch <branchname> //新建分支
	
	git branch -d <branchname> //删除本地分支
	
	git branch -d -r <branchname> //删除远程分支，删除后还需推送到服务器
	git push origin:<branchname>  //删除后推送至服务器
	
	git branch -m <oldbranch> <newbranch> //重命名本地分支
	/**
	*重命名远程分支：
	*1、删除远程待修改分支
	*2、push本地新分支到远程服务器
	*/
	
	//git中一些选项解释:
	
	-d
	--delete：删除
	
	-D
	--delete --force的快捷键
	
	-f
	--force：强制
	
	-m
	--move：移动或重命名
	
	-M
	--move --force的快捷键
	
	-r
	--remote：远程
	
	-a
	--all：所有

	git check -b <新分支名称> //新建分支并切换到新建分支

	git branch -vv // 查看本地处于远程库中哪个
	```
2. git 操作远程分支

把新建的本地分支push到远程服务器，远程分支与本地分支同名（当然可以随意起名）：`git push origin <本地分支名>:<远程分支名>`,删除远程已有分支：`git push origin --delete dbg_lichen_star`

```bash
$ git branch -a
* dev
  master
  remotes/origin/dev
  remotes/origin/master

推送到远程：
$ git push origin dev:dev2
Total 0 (delta 0), reused 0 (delta 0)
remote:
remote: Create a pull request for 'dev2' on GitHub by visiting:
remote:      https://github.com/runooboy/runooboy.github.io/pull/new/dev2
remote:
To github.com:runooboy/runooboy.github.io.git
 * [new branch]      dev -> dev2

结果：
$ git branch -a
* dev
  master
  remotes/origin/dev
  remotes/origin/dev2
  remotes/origin/master

删除远程已有分支：
$ git push origin --delete dev2
To github.com:runooboy/runooboy.github.io.git
 - [deleted]         dev2

结果：
$ git branch -a
* dev
  master
  remotes/origin/dev
  remotes/origin/master
```











