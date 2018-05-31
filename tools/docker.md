---
title: docker
date: 2018-5-30 15:17:00
tags: [docker]
---

# 安装

win10 不能直接安装Docker，需要安装一个[Docker Toolbox](https://docs.docker.com/toolbox/toolbox_install_windows/)

安装完成后到安装目录下运行`start.sh`进入虚拟环境。

Kitematic (Alpha)是Docker图形化工具。

# 常用指令

查看下载的docker image
```
docker image ls
```

查看由image生成的container
```
docker container ls --all
```

使用列出的id开启container
```
docker start xxxxxxxx
```

以及停止、删除等操作，具体可通过help查到
```
docker container --help
```

由image创建一个contaner并启动，-i表示交互模式，-t表示使用TTY，--name表示给容器命名，redis为指定的image名称
```
docker run -it --name redis-test redis
```

其他操作可以--help查看
```
docker run --help
```
