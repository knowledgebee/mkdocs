---
title: 使用 Git
date: 2022-02-25 22:28:47
tags:
    - Software
    - Encryption
---

如何使用开源版本控制系统Git

<!--more-->

Git 是一个开源的版本控制系统，可以很方便的对文件进行版本控制。Git 的版本控制是对文件进行快照记录，记录文件的差异和修改，因此相对与 SVN 在文本代码等场景在占用空间方面比较有优势。

在开始使用 Git 之前需要设置 Git 的用户名和邮箱：

> *git config --global user.name "\<Your UserName\>"*
> *git config --global user.email "\<Your Email\>"*

Git 在开始跟踪文件的版本变化前要先建立一个 Git 的代码仓库，Git 初始化代码仓库的命令为：

> *git init*

该命令将在当前文件夹下新建一个 Git 代码仓库。在代码仓库建立完成后代码仓库还没有跟踪任何文件，要对文件进行版本控制需要将文件添加至代码仓库进行跟踪，将文件添加至当前 Git 代码仓库的命令为：

> *git add \<Filename\>*

该命令将 *`<Filename>`* 添加至 Git 在当前文件夹下的代码仓库的暂存区，如果需要添加文件下的所有文件可以将 *`<Filename>`* 替换为 *`<-A>`*,此时文件还未添加在 Git 的代码仓库中，要将保存在暂存区的文件添加在 Git 仓库并建立索引快照需要“提交”动作( *`git commit`* )：

> *git commit -m "Massage For This commit"*

该命令中的 *`-m`* 参数表明后面双引号中的内容为这次提交的信息，如果没有该参数和信息将在提交时弹窗编辑提交信息。

以上操作都是在本地进行的，并不会对远程仓库造成任何影响，如果要将本地仓库与远程仓库同步，需要先在本地添加远程仓库的地址：

> *git remote add \<Local Mark\> \<Remote Repo Url\>*

add后面跟的是远程仓库在本地的标识，一个本地仓库可能与多个远程仓库同步，设置一个简短有特点的本地标识对提升效率有所帮助。需要查看本地仓库添加的全部远程仓库使用:

> *git remote -v*

删除本地仓库添加的远程仓库：

> *git remote remove \<Local Mark\>*

将本地仓库与远程仓库完全同步:

> *git push -u \<Local Mark\>*

查看提交历史记录：

> *git log --oneline*

该命令会显示提交历史记录和该提交的哈希值，用哈希值可以让工作目录回到回到某次提交的状态：

> *git checkout \<Hash For commit\>*

回到最后一次提交的状态,将 Header 切换到分支即可：

> *git checkout \<Branch Name\>*

Git支持对每次的提交进行签名，如果需要对每次的提交进行签名：

> *git config --global user.signingkey \<Your GPG key Fingerprint\>*
>
> *git config --global commit.gpgsign true*
