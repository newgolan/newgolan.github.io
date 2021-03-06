---
layout:     post
title:      "GIT常用命令"
subtitle:   "The Systematic Failure of Higher Education in China"
date:       2021-01-19 12:00:00
author:     "Axming"
catalog: true
header-style: text
tags:
  - GIT
---

github是一个基于git的代码托管平台， Git 并不像 SVN 那样有个中心服务器。
目前我们使用到的 Git 命令都是在本地执行，如果你想通过 Git 分享你的代码或者与其他开发人员合作。
你就需要将数据放到一台其他开发人员能够连接的服务器上。 

1.常用

ssh-keygen -t rsa -C "zmy112334@163.com"		登录仓库 回车

git clone https://github.com/xxx	克隆仓库到本地

git pull		远程更新

git add   （文件名）| /*  提交到文件夹    （文件名） 为单个文件 将本地文件提交到暂存区  

git add ./		添加目录下文件

git commit -m "代码提交信息"		提交到本地仓库

git status	查看状态

git push	为从暂存区提交到远程仓库

git checkout		更改分支

git branch		查看分支

git push origin :feature		删除远程分支

2.基本操作

mkdir filename		创建目录

pwd		显示当前目录的路径

git init 	把当前的目录变成可以管理的git仓库，生成隐藏.git文件

git diff filename.*		查看filename文件修改了那些内容

git log		查看历史记录

git reflog		查看历史记录的版本号id

git reset --hard 版本号		回退至目标版本号

git remote add origin https://github.com/xxx	关联一个远程库

git branch 分支名		创建分支

git merge dev		在当前的分支上合并dev分支

git branch –d dev		删除dev分支

git remote		查看远程库的信息

git remote –v		查看远程库的详细信息

git push origin master		Git会把master分支推送到远程库对应的远程分支上


[张明远]:      https://zmy1123347389.github.io/