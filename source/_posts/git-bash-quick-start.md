---
title: Git Bash 的常用方法
date: 2018-03-21 14:44:03
tags:
 - 工具
---

之前一直用GUI，现在想简单点用git bash，整理git的常用命令。  

### 基础知识  
![](/assets/blogImgs/git-version-manage.jpg)  

git是分布式版本控制软件。  git文件夹内的.git文件夹是仓库repository，其他是文件或者目录是working tree。  
repository内包含暂存区stage、分支branch（默认会有master分支，有一个名为master的指针指向该分支的当前commit）以及指向指针master的指针HEAD。  

![](/assets/blogImgs/git-branch.jpg)  
branch代表的就是由多个连续的状态点变迁串联起来的抽象概念，而每个branch都有一个同名指针指向当前commit。  

### 基础命令  

```bash
$ git init  
```
初始化当前目录为Git仓库。  

```bash
$ git add <file>
```
从working tree添加不定数量的file到stage。 

```bash
$ git commit -m <msg>
```
将stage的所有内容commit到当前分支，stage将被清空。  

```bash
$ git diff
```
比较working tree与stage中现有内容异同。

```bash
$ git status
```
查看当前branch各方面状态，很重要。  
  
```bash
$ git log [--graph]
```
查看commit历史，加上--graph可以看到时间线，记录中最重要的是commit id以及相应commit msg
  
```bash
$ git branch
```
查看当前HEAD指向，列出所有branch。  

```bash
$ git checkout <branchname>
$ git checkout -b <branchname>
```
第一条，从当前分支切换到branchname，具体内容是将branchname当前commit的内容复制到working tree上，清空stage,并将HEAD指向branchname。  
第二条，创建并切换到branchname，具体内容是repository创建一个指针branchname指向当前分支的当前commit，并将HEAD指向branchname。  

**注意**
切换分支的时候要保证working tree clean && nothing to commit，否则会比较麻烦。  
假设当前分支develop下的working tree有改动，又不想commit，想直接去别的分支解决问题，则可以使用
```bash
$ git stash
```
将working tree现场暂存起来，恢复working tree clean的状态，然后再去别的分支解决问题。解决问题之后回到master，使用
```bash
$ git stash pop
```
恢复working tree现场，继续工作。  

**注意注意**
stash虽好可不要贪杯，因为stash是所有分支共用的，如果在两个分支都进行了stash操作，也就意味着stash list栈里有两个分支的working tree现场，这个时候要使用
```bash
$ git stash list
```
查看到底哪一个stash是是属于当前分支的，辨别好后，手动使用
```bash
$ git stash apply stash@{n}
```
恢复现场，而且确定恢复正确后最好使用
```bash
$ git stash drop stash@{n}
```
删除该stash。如果回复现场用错了stash@{n}，那你会很苦恼，因为这又涉及到merge了，不要走到这一步，很麻烦。  
最好不要多分支stash，不然stash就失去了其便利的意义了：P  


### 后悔药  
```bash
$ git checkout -- <file>
```
撤销working tree中对应file modification，具体内容是用当前commit中的file替换working tree的file。  

```bash
$ git reset HEAD <file>
```
unstage，撤销已经add到stage中对应的file modification，具体内容是用当前branch最新commit中的对应的file替换stage的file。
  
```bash
$ git reflog
```
列出所有的version change，最主要是commit id，穿梭时光利器！   


```bash
$ git reset [--hard | --soft] [commit id | HEAD^ | HEAD^^]
```
将当前branch版本重置到目标commit（HEAD表示当前commit，HEAD^表示上个commit，HEAD^^表示上上个commit）。  
--hard参数会将working tree、stage的内容重置为目标版本内容。  
--soft参数会使得working tree、stage保持原样。  

```bash
$ git rm file
```
这个有点糊涂，之后再试作用。。。  

### 分布式大法  

配合支持git的代码托管平台使用，比如github，在github上创建相同名字的repository。  
```bash
$ git remote add origin git@github.com:username/reponame.git  
```
给local repository关联remote repository，并将remote repository命名为origin。  

```bash
$ git push -u origin master
$ git [push | pull] origin master
```
第一条，将local repository的branch master的内容push到remote，-u参数将local master和remote master关联起来，第二条，执行第一条以后直接使用push或pull。  

```bash
$ git pull
```
将remote分支的内容拉下来，和local分支进行merge，同时将merge结果checkout到working tree。  
**注意**  

如果local分支的working tree 有改动没有add、commit，这时pull有两种情况：
- pull的结果不会导致overwrite the unadded modification of working tree，顺利pull，这时进行了merge，这时merge分为两种情况；  
	- fast-forward模式，在local分支与remote分支处在同一commit链上。这时只有remote分支走在local分支前面的一种情况。因为local走在remote前面时，pull操作会提示already up to date。
	- no-ff模式，local分支与remote分支不在一条链上（也就是local和remote各自commit过），有可能需要手动解决coflict。
- pull的结果will overwrite the unadded modification of working tree, pull aborted。这时有两个选择：  
	- stash the modification of working tree，然后pull，最后把stash pop出来,pop出来之后手动解决conflict。
	- commit the modification of working tree,然后pull，最后手动解决conflict。

**好的习惯是先在github上create repository，然后clone下来开发**  

```bash
$ git clone git@github.com:username/reponame.git
$ git clone -b branch git@github.com:username/reponame.git /path
$ git branch -a
$ git checkout -b develop origin/develop
```
git clone会clone整个remote repository，但在local默认只创建master分支，其他分支可以指定创建。  

### 分支--分治--融合--删除  

#### 分治  

repository一般有分支master、acceptance、develop、feature、issue  
master：主线分支，代码稳定，用于正式版本发布。  
acceptance：测试分支，代码基本稳定，准master分支。  
develop：开发分支，代码不稳定，用于日常开发，团队成员往里面merge阶段性开发代码。  
feature：特性分支，用于开发项目某个具体特性，然后merge到develop。  
issue：debug分支，用于调试修改bug，然后merge到devlop分支。  


#### 融合
merge有两种模式fast-forword、no-ff：  
- fast-forword模式会直接将master指向develop，no-ff模式会merge两个分支的commit形成新的conmmit，并将master指向新的commit。  
- no-ff模式merge时会遇到两种情况：有冲突、无冲突。  

假设当前所在分支为master，需要merge develop到master。  

无冲突情况存在于两种分支状态：  
- develop的当前commit的前一个版本是master的当前commit，同时只有这种情况适合fast-forword模式。

- develop的commit的前一个版本不是master的当前commit，但是两个commit之间的diff不会相互干扰，换句话说就是两个commit之间的diff不会出现在同一个文件里。  
有冲突的情况存在于：  
- develop的commit的前一个版本不是master的当前commit，同时两个commit之间的diff相互干扰，换句话说就是两个commit之间的diff会出现在同一个文件里。  
在有冲突的merge中需要手动处理冲突，然后手动commit。  


```bash
$ git merge <--no-ff> develo
```
不加参数--no-ff，并满足以上所述条件,则默认是fast-forward模式，就是直接将master指向develop的当前commit。fast-forward模式没有merge历史分支。  
加上参数--no-ff，并满足以上所述条件，则是no-fff模式，无冲突情况下master自动commit，但依然需要手动写commit msg；有冲突情况下需要手动解决冲突、手动commit。no-fff模式有merge历史分支。    
merge只会改变master，不会改变develop的状态。merge操作会使得working tree、stage与merge commit保持一致。

#### 删除  
```bash
$ git branch <name>
$ git branch -d <name>
```
git鼓励create branch来解决问题，merge之后删除branch，这是一种安全高效的方式。  

### TODO  
git的stage 这块的具体机制没搞清楚，index什么的之后看书搞清楚吧。。。。。  
