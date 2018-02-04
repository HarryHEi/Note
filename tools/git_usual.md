---
title: git常用
date: 2016-03-24 21:04:06
tags: [git]
---

# 简介
git本地仓库由三个东西构成:
+ 工作目录
+ 暂存区
+ HEAD
## git init
切换到工作目录，使用git init初始化仓库
## git config
信息配置
```
git config --global user.name "name" 
git config --global user. email "somebody@any.com"
git config --list  #列出配置
```
## git help
获得帮助，后面可以添加参数
```
git help commit
```
## git add <filename>
```
git add <filename>
```
将文件添加到暂存区，可以使用
```
git add . 
```
添加所有文件
## git mv <filename> <newname>
```
git mv <filename> <newname>
```
使用mv指令更改文件名或者移动文件
## git rm <filename>
```
git rm <filename>
```
删除文件
## git commit -m "some message"
```
git commit -m "some message"
```
将改动提交到HEAD，并标注一段信息
```
git commit -am "some message"
```
加上a将会更新暂存区内容并commit
## git log
查看commit id
```
git log --author="name"
```
查看特定author的提交信息
## git status
查看工作区状态
如果工作区有改动，会和暂存区对比并显示
## git diff
显示工作区和暂存区的不同点
```
git diff --staged
```
加入参数staged，能够显示与HEAD之间的不同点
## git checkout <filename>
```
git checkout <filename>
```
将某个文件回滚到暂存区的状态
```
git checkout "commit number" -- "filename"
```
新分支整合到主分支后面
```
git rebase
```
给某个版本打上标签
```
git tag ***
```
将某个文件回滚到某个commit版本
## git reset HEAD <filename>
```
git reset HEAD <filename>
```
将某个文件重置到HEAD里面的版本，注意，后面的版本会被删除
## git remote add <name> <your url>
```
git remote add <name> <your url>
```
连接远程仓库
## git remote -v
查看连接的远程仓库
## git push <name> <localbranch>:<obj branch>
```
git push <name> <localbranch>:<obj branch>
```
```
git branch -a #查看所有分支
git branch -r #查看远程分支
```
本地HEAD推送到远程仓库
```
git push HarryHEi master:hexo
```
## git pull <name> <obj branch>:<local branch>
```
git pull <name> <obj branch>:<local branch>
```
更新本地文件
## .gitignore
创建一个名为.gitignore的文件，push无视某些文件

## git bisect
http://www.oschina.net/translate/letting-git-bisect-help-you
使用二分伐查找问题版本

```
#开始
git bisect start

#如果当前版本为bug版本，就设置为bad
git bisect bad

#查看最一开始版本号
git log | tail -n -5  
#设置最开始版本为good
#这时候会自动切换到中间版本，并返回版本号
git bisect good

#检查版本，并继续打上标记
git bisect good
git bisect bad

直到找到问题版本
```

# git设置proxy

```
git config --global http.proxy 127.0.0.1:8787
```

# git取消proxy

```
git config --global --unset http.proxy
```

```
$ git push origin master
fatal: unable to access 'https://github.com/HarryHEi/HarryHEi.github.io.git/': Could not resolve host: github.com
```
<!-- more -->
考虑到可能是ssh key的问题，然后重新设置key
```
ssh-keygen -t rsa -b 4096 -C "harryx520@qq.com"
cd /home/herui/.ssh/
```
复制id_rsa.put内容到github，添加ssh key
连接
```
ssh -T git@github.com
```
然后报错
```
ssh: Could not resolve hostname github.com: Name or service not known
```
又可能是host设置的问题
打开 /etc/hosts
```
$ sudo vim /etc/hosts
```
添加一行
```
192.30.252.130     github.com
```
连接
```
ssh -T git@github.com
```
输入密码，设置完毕
push 成功
```
$ git push origin master
Everything up-to-date
```

# 参考资料
<http://blog.csdn.net/ithomer/article/details/7529022>
<https://www.youtube.com/watch?v=cEGIFZDyszA&index=1&list=PL6gx4Cwl9DGAKWClAD_iKpNC0bGHxGhcx>
