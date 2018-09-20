---
title: Git常用操作
date: 2018-08-26 23:22:00
categories:
- Git
tags:
- command
---

gitee 代码托管

# Git常用操作

> 目录

- 本地推送代码到服务器
- 常见问题
- git配置、操作命令补充

## 本地代码推送到服务器

> 操作前提

- 配置用户和密码
- 配置公钥
- 代码托管平台创建项目

> 推送顺序

- [1] 新建文件夹

  `mkdir spring-boot-samples`
  `cd spring-boot-samples`

- [2] 初始化目录

  `git init`

- [3] 添加待提交文件

  `git add .`

- [4] 添加提交描述

  `git commit -m "commit spring-boot-samples"`

- [5] 配置远端地址(需要在线创建新项目)

  `git remote add origin git@gitee.com:xiacy/spring-boot-samples.git`

- [6] 推送文件到远端服务器

  `git push origin master`

> 本地和远端文件不一致导致推送失败（远端有，但本地没有）、本地文件托管到新建项目时

```
xiazhaoyangdeMacBook-Pro:spring-boot-samples xiazhaoyang$ git push origin master
To gitee.com:xiacy/spring-boot-samples.git
 ! [rejected]        master -> master (fetch first)
error: failed to push some refs to 'git@gitee.com:xiacy/spring-boot-samples.git'
hint: Updates were rejected because the remote contains work that you do
hint: not have locally. This is usually caused by another repository pushing
hint: to the same ref. You may want to first integrate the remote changes
hint: (e.g., 'git pull ...') before pushing again.
hint: See the 'Note about fast-forwards' in 'git push --help' for details.

```
- [7] 更新本地差异文件

  `git pull --rebase origin master`

```
xiazhaoyangdeMacBook-Pro:spring-boot-samples xiazhaoyang$ git pull --rebase origin master
warning: no common commits
remote: Enumerating objects: 74, done.
remote: Counting objects: 100% (74/74), done.
remote: Compressing objects: 100% (41/41), done.
remote: Total 74 (delta 6), reused 0 (delta 0)
Unpacking objects: 100% (74/74), done.
From gitee.com:xiacy/spring-boot-samples
 * branch            master     -> FETCH_HEAD
 * [new branch]      master     -> origin/master
First, rewinding head to replay your work on top of it...
Applying: commit spring-boot-samples
Using index info to reconstruct a base tree...
.git/rebase-apply/patch:546: trailing whitespace.
## Spring Boot
.git/rebase-apply/patch:581: trailing whitespace.

.git/rebase-apply/patch:5340: trailing whitespace.

.git/rebase-apply/patch:5344: trailing whitespace.

.git/rebase-apply/patch:5346: trailing whitespace.

warning: squelched 4 whitespace errors
warning: 9 lines add whitespace errors.
Falling back to patching base and 3-way merge...
Auto-merging .idea/spring-boot-samples.iml
CONFLICT (add/add): Merge conflict in .idea/spring-boot-samples.iml
Auto-merging .idea/modules.xml
CONFLICT (add/add): Merge conflict in .idea/modules.xml
error: Failed to merge in the changes.
Patch failed at 0001 commit spring-boot-samples
Use 'git am --show-current-patch' to see the failed patch

Resolve all conflicts manually, mark them as resolved with
"git add/rm <conflicted_files>", then run "git rebase --continue".
You can instead skip this commit: run "git rebase --skip".
To abort and get back to the state before "git rebase", run "git rebase --abort".

```

- 重复步骤[2-4]、[6-7]根据提示解决文件差异

```
xiazhaoyangdeMacBook-Pro:spring-boot-samples xiazhaoyang$ git pull --rebase origin master
From gitee.com:xiacy/spring-boot-samples
 * branch            master     -> FETCH_HEAD

It seems that there is already a rebase-apply directory, and
I wonder if you are in the middle of another rebase.  If that is the
case, please try
	git rebase (--continue | --abort | --skip)
If that is not the case, please
	rm -fr "/Users/xiazhaoyang/Capsule/workspace/spring-boot-samples/.git/rebase-apply"
and run me again.  I am stopping in case you still have something
valuable there.

```

  `git rebase --skip`
  `git push origin master`

```
xiazhaoyangdeMacBook-Pro:spring-boot-samples xiazhaoyang$ git rebase --skip
xiazhaoyangdeMacBook-Pro:spring-boot-samples xiazhaoyang$ git push origin master
Counting objects: 334, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (260/260), done.
Writing objects: 100% (334/334), 181.34 KiB | 4.42 MiB/s, done.
Total 334 (delta 78), reused 0 (delta 0)
remote: Resolving deltas: 100% (78/78), done.

```

> 推送结果

![输入图片说明](https://images.gitee.com/uploads/images/2018/0920/082529_7c141bf3_912956.png "屏幕截图.png")

## 常见问题

>  Git Credential Manager for windows 用户名和密码输入无法通过，导致推送代码失败

`解决方法`：

执行 `git config --system --unset credential.helper`

执行这个命令之后，你可以重新写入账号密码，这样就可以重新提交代码了。

---

> 下面补充介绍下`git`配置、公钥生成以及项目常用的一些git操作

## 配置用户信息

> 命令行设置

```
$ git config --global user.name "Your Name"     配置用户名
$ git config --global user.email "email@example.com"  配置邮件
```

## 生成并部署SSH key

> 生成 sshkey:

```
ssh-keygen -t rsa -C "xxxxx@xxxxx.com"  
# Generating public/private rsa key pair...
# 三次回车即可生成 ssh key
```

    查看你的 public key，并把他添加到码云（Gitee.com）或 github （github.com） 的SSH上

```
cat ~/.ssh/id_rsa.pub（公钥的文件）
# ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC6eNtGpNGwstc....
```

添加后，在终端（Terminal）中输入

```
ssh -T git@git.oschina.net
```

若返回
    Welcome to Git@OSC, yourname!
则证明添加成功。

## 项目中常用的git操作

### 项目操作

- 克隆项目

    git clone url(一般用ssh格式)
    git pull

- 添加自己修改的文件

    git add . 修改过的所有文件添加到git仓库
    git commit
    进入vim编辑，写注释
```
1、k,j,h,l   光标上下左右 （也可以使用上下左右键）                      
2、o	       在当前行下方插入新行并进入插入模式 (一般写注释，注释要详细)                    
3、:wq	   保存并退出
注意：注释添加完毕后需要先将输入法切换为英文并按ESC键输入:wq指令后按Enter  
```

- 提交代码

    git push

>注：

    >1 先git pull 下载解决冲突（必要时手动解决冲突）再git push  提交服务器
    >2 svn的流程与git相似，但是svn是可视化的工具，也是分为三步走:
    新建文件夹克隆代码-->svn updata-->svn commit
    如果有不想提交的文件可以使用：.gitignore文件（如node_modules的提交）
    详细文档请查看：https://git.oschina.net/hjm100/codes/cqylm4gubre1f9ax23hw594

    >3 git commit	标准用法：
		不推荐使用 -m 简单模式
		推荐使用编辑器提供详细的提交信息：
		提交内容概要
		<----注意此处必须有空行---->
		提交内容1
		提交内容2
		提交内容3
		...
    【注意：合并者要保证代码格式，否则会被别人鄙视！】

## git常用指令
```
git init                          在本地初始化一个Git库
git clone <url>                   将远程服务器上的项目文件夹包括.git隐藏文件丝毫不差地下载到本地
git add file                      添加文件到git库
git commit -m                     添加注释，一般可以是总结做的事情
git commit --amend                使用 --amend 可以修补提交消息（可在修改后先 git add 再 git commit --amend 修补刚才的提交）
git pull                          从远程git库中拉取最新内容并合并，它相当于 git fetch 和 git merge
git push                          将本地git库推送到远程git库
                                  推送时有2种可能：
		                          1、远程git库自从上次pull之后没有发生过变化快速向前自动完成
		                          2、远程git库已经被其他人push了新内容
git push origin blue -u           将本地分支推送到远程git库（origin）-u 将本地分支和远程git库中的blue分支关联起来
git push <远程git库名称> --tags    将所有标签推送到远程git库
git remote                        查看与本地git库关联所有远程git库（一个本地git库可以向多个远程git库推送）
git remote -v                     查看远程库的url及权限（verbose详细信息）
git remote show <远程库名字>       可以查看远程的基本信息（如：主分支名字，全部分支的列表，本地库和远程库之间的差异）          
git tag <标签名>	                在最近的提交上打标签
git show <标签名>	                显示标签信息（标签打在了哪次提交上）
git tag		                       列出所有标签
git tag -d <标签名>	             删除标签
git tag -a <标签名> -m "打标签的原因"  新增附注标签，附加有备注信息及打签人和打签时间的标签
git log                              查看历史提交，方便回退
git status                           告诉你工作区的当前状态
git diff                             查看修改内容
git config --global -l               查看全局配置（当前用户目录中的  .gitconfig  文件）
git reset --hard  版本id             回退到某一个版本
git  reflog                          查看命令历史
git rm file                          从版本库中删除一个文件
```
>注意：注意如果不提交，这次删除不会保存在git库中，想要保存需要提交commit。
文件删除之后，通过checkout 找不回这个文件，如果想要找到这个文件采用回退命令。
```
git checkout -- file                 从git库中获取一个文件版本替换工作区文件版本。
git checkout -b 分支名               如果分支不存在创建新分支， 然后切换到这个分支
工作区（work directory）、暂存区（stage）、master
修改git默认编辑的方法：
 git config --global core.editor "编辑器名字或路径 -w"   -w 表示git需要等待编辑器窗口关闭再提交
```


## git合并命令以及vim编辑器

```
git log		        打印提交记录，如果显示过多，按回车继续打印，按q退出日志打印
git merge 分支名    合并指定分支到当前分支，有多种分支合并模式，Fast Forward最简单，最快速，不需要人参与
echo 内容>>文件名    创建文件，并将内容注入到文件中
vim 文件名		     使用vim编辑文件，命令行编辑器
h			        光标向前
l			        光标向后
j			        光标向下
k			        光标向上
i			        进入插入模式（输入模式）默认vim进入的是命令模式，命令模式下按键时执行命令，插入模式下按键插入字符
:wq			        保存并退出
:q!			        不保存退出
o			        在当前行下方插入新行并进入插入模式
vimtutor		    vim入门教程
v			       可视化选区
d			       删除可视化选区并记录删除的内容
p			       粘贴内容
u			       撤消

合并冲突、解决冲突实际上就是人工修改代码

git log --graph		以“图形”化的方式显示提交记录,可以很方便地查看各分支提交情况

```

## git分支操作

```
在本地新建一个分支： git branch newBranch
切换到你的新分支: git checkout newBranch
将新分支发布在github上： git push origin newBranch
在本地删除一个分支： git branch -d newBranch
在github远程端删除一个分支： git push origin :newBranch (分支名前的冒号代表删除)
```

> 备注：在Git操作中你不需要了解太多指令，只需要熟记项目中常用的指令即可，希望有兴趣的朋友，可以自己操作上手一下（可以搭建自己的项目）熟能生巧，读万卷书不如行万里路，谢谢！！！！

## REFRENCES

1. [Git 添加一个文件夹, 并推送到gitee, 步骤](https://blog.csdn.net/jkihi_lin/article/details/79974712)
2. [鸿基梦 / Git操作总结.txt](https://gitee.com/hjm100/codes/dgbpt1lfnuvjqor52awcs80)

---

## 微信公众号

<center>
<img src="https://images.gitee.com/uploads/images/2018/0717/215030_8e782063_912956.png" width="50%" height="50%"/>
</center>

扫码关注或搜索`架构探险之道`获取最新文章，不积跬步无以至千里，坚持每周一更，坚持技术分享。我和你们一起成长 ^_^ ！
