---
title: Git Bash 的常用方法
date: 2018-03-21 14:44:03
tags:
 - 工具
---

之前一直用GUI，现在想简单一点直接用git bash，整理git的常用命令。  

### 基础知识   
![](/assets/blogImgs/git-version-manage.jpg)  

git是分布式版本控制软件。  git文件夹内的.git文件夹是仓库```repository```，其他是文件或者目录是```working tree```。  
在```repository```内包含暂存区```stage```、分支```branch```（默认会有master分支）以及指向分支的指针```HEAD```。  

### 基础命令  

```  
$ git init  
```  
初始化当前目录为Git仓库。  

```  
$ git add <file> 
```  
从working tree添加不定数量的file到stage。 

```  
$ git commit -m 'change log'  
```  
将stage的所有内容commit到当前分支，stage将被清空。  

```  
$ git diff  
```  
比较working tree与stage中现有内容异同。

```  
$ git status  
```  
查看当前branch各方面状态，很重要。  
  
```  
$ git log [--graph]
```  
查看commit历史，加上--graph可以看到时间线，记录中最重要的是commit id以及相应commit msg
  
```  
$ git branch
```  
查看当前HEAD指向，列出所有branch。  

```
$ git checkout branch  
$ git checkout -b branch
```  
第一条，切换branch，具体内容是将branch最新commit的内容复制到working tree,并将HEAD指向branch的最新commit上。  
第二条，创建并切换branch，具体内容是将working tree内容commit到新创建的branch，并将HEAD指向新创建的branch的最新commit上。  

### 后悔药  
```  
$ git checkout -- file
```  
撤销working tree中对应file modification，具体内容是用stage对应的file替换working tree的file。  

```  
$ git reset HEAD file
```  
unstage，撤销已经add到stage中对应的file modification，具体内容是用当前branch最新commit中的对应的file替换stage的file。
  
```  
$ git reflog  
```  
列出所有的version change，最主要是commit id，穿梭时光利器！   


```  
$ git reset [--hard | --soft] [commit id | HEAD^ | HEAD^^]  
```  
将当前branch版本重置到目标commit id状态。（HEAD表示当前版本，HEAD^表示上个版本，HEAD^^表示上上个版本）。  
hard参数会将working tree、stage的内容重置为目标版本内容。  
soft参数会使得working tree、stage保持原样。  

```  
$ git rm file
```  
这个有点糊涂，之后再试作用。。。  

### 分布式大法  
![]()  

配合支持git的代码托管平台使用，比如github，在github上创建相同名字的repository。  
```  
$ git remote add origin git@github.com:username/reponame.git  
```  
给local repository关联remote repository，并将remote repository命名为origin。  

```  
$ git push -u origin master  
$ git [push | pull] origin master
```  
第一条，将local repository的branch master的内容push到remote，-u参数将local master和remote master关联起来，第二条，执行第一条以后直接使用push或pull。  

**好的习惯是先在github上create repository，然后clone下来开发**  

```  
git clone git@github.com:username/reponame.git  
```  
克隆仓库  

### 分支--分治  
**repository一般有master、acceptance、develop、feature、issue**  s

```master```：主线分支，代码稳定，用于正式版本发布。  
```acceptance```：测试分支，代码基本稳定，准master分支。  
```develop```：开发分支，代码不稳定，用于日常开发，团队成员往里面merge阶段性开发代码。  
```feature```：特性分支，用于开发项目某个具体特性，然后merge到develop。  
```issue```：debug分支，用于调试修改bug，然后merge到devlop分支。  



