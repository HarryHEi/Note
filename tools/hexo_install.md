---
title: 懒癌归来
date: 2017-03-14 21:54:02
tags: [hexo]
---

由于环境搬迁这个博客很烦，自从换了电脑之后一直没更新（其实就是懒），时隔四个月，折腾了一个小时，这次终于成功克服懒癌搬回来了^ ^.  好好学习天天向上咯~ 下面是整理的几个坑。

# Windows安装时的几个坑

github注册，新建仓库什么的就跳过了，然后是git的安装，在官网下载git的速度简直了，好在github有同步更新的网站。[网址](https://github.com/waylau/git-for-win)下载完成后安装，不要忘了设置环境变量。

然后是node.js的安装，这里到官网下载安装，没什么问题。

然后用npm安装hexo又不动了，需要使用淘宝的源

```
npm install -g cnpm --registry=https://registry.npm.taobao.org
```

然后后面就把所有的npm命令换成cnpm，后面没什么问题。

```
cnpm install hexo-cli -g
d:
hexo init blog
cd blog
cnpm install
```

后面就是缺什么就用`cnpm`装什么

是hexo初始化安装什么的还算顺利，配置deploy:

```
deploy:
  type: git
  repository: https://github.com/HarryHEi/HarryHEi.github.io.git
  branch: master
  message: update
```

在`hexo d`的时候有时候会一直没反应，用管理员运行cmd，多试几次又可以了。
