---
title: Git Bash 的常用方法
date: 2018-03-21 14:44:03
tags:
 - 工具
---
![](/assets/blogImgs/git-version-manage.jpg)

## Git Bash 的版本管理   

一个新创建的git文件夹中分为working directory和repository  
working directory里的.git文件夹就是repository  
repository有暂存区stage、自动创建的分支master、指向master的指针HEAD  

### 基础命令  

```  
$ git init  
```  
初始化当前目录为Git仓库  

```  
$ git add <file> 
```  
从working directory添加不定数量的文件到stage 

```  
$ git commit -m 'change log'  
```  
将stage的所有内容添加到当前分支  

```  
$ git diff  
```  
查看working directory与stage内容的异同

```  
$ git status  
```  
查看woking directory、stage、branch之间的不同  

### 后悔药  
```  
time machine start...  
```  
