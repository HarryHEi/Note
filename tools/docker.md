---
title: docker
date: 2018-5-30 15:17:00
tags: [docker]
---

# 常用命令
## 镜像
### 获取镜像
```
docker [image] pull

例：获取ubuntu镜像：docker pull ubuntu:18.04
```

### 列出镜像
```
docker image ls 或者 docker images
```
### 添加标签
```
例：docker tag monitor:latest monitor:0.1
```

### 查看镜像信息
```
docker [images] inspect

例：docker inspect ubuntu:18.04
```

### 查看镜像层级历史
```
docker history 

例：docker history ubuntu:18.04
```

### 搜索镜像
```
docker search

例：docker search nginx
```

### 删除镜像
```
docker rmi 或者 docker image rm

例：docker rmi ubuntu:18.04
```

### 清理镜像
```
docker image prune
```

### 根据容器创建镜像
```
docker [container]  commit CONTAINER IMAGE:TAG

例：docker commit mymonitor monitor:0.1
```

### 从本地操作系统模板导入镜像
```
docker [image] import file | URL - IMAGE:TAG

例：cat ubuntu-18.04-x86_64-minimal.tar.gz | docker import - ubuntu:18.04
```

### 基于`Dockerfile`创建镜像
```
docker [image] build -t IMAGE:TAG .

例：docker build -t monitor:2.0 .
```

### 导出镜像
```
docker [image] save

例：docker save -o ubuntu_18.04.tar ubuntu:18.04
```

### 载入镜像
```
docker [image] load

例：docker load -i ubuntu_18.04.tar 或者 docker load < ubuntu_18.04.tar
```

### 上传镜像
```
docker [image] push

例：docker push user/test:latest
```

## 容器
### 创建容器
```
docker [container] create

可选参数
    -i 保持标准输入打开
    -t 是否分配一个伪终端
    -d 是否在后台运行容器
    -p 如何映射本地端口
    --name 指定容器别名
    --rm 退出后删除

例：docker create -it ubuntu:18.06
```

### 启动容器
```
docker [container] start

例：docker start monitor
```

### 创建并启动容器
```
docker [container] run
```

### 暂停容器
```
docker [container] pause
```

### 终止容器
```
docker [container] stop
```

### 清理停止状态的容器
```
docker container prune
```

### 启动容器
```
docker [container] start
```

### 重启容器
```
docker [container] restart
```

### 进入正在执行的容器
```
docker [container] attach

退出使用 CTRL-P  CTRL-Q
```

### 在容器内执行命令
```
docker [container] exec [-d] [-i] [-t] CONTAINER COMMAND

例：docker exec -it monitor /bin/bash
```

### 删除容器
```
docker [container] rm
```

### 导出容器
```
docker [container] export

例：docker export -o monitor.tar monitor 或者 docker export monitor > monitor.tar
```

### 导入容器
```
docker [container] import file | URL | - IMAGE:TAG

导入的容器会变为镜像，例：docker import monitor.tar - monitor:v1.0
```

### 查看容器详情
```
docker container inspect CONTAINER [CONTAINER …]

例：docker container inspect monitor1 monitor2
```

### 查看容器内进程
```
docker [container] top CONTAINER [CONTAINER …]
```

### 查看容器资源统计信息
```
docker [container] stats [CONTAINER …]
```

### 复制文件
```
docker [container] CONTINER:SRC_PATH DEST_PATH

例：docker cp data monitor:/tmp/
```

### 查看容器端口映射
```
docker container port CONTAINER
```

### 更新容器配置
```
docker [container] update CONTAINER
```

## 数据卷
### 创建数据卷
```
docker volume create NAME
```

### 其他数据卷操作
```
inspect（查看详细信息）ls（列出已有的数据卷）rm（删除数据卷）prune（清理无用的数据卷）
```

### 容器绑定数据卷
```
例如：docker run  -it --mount type=bind,source=/work/zbx,destination=/opt/zbx --name monitor monitor /bin/bash

等同：docker run -it -v /work/zbx:/opt/zbx --name monitor monitor /bin/bash
```

# `Dockerfile`
## 常见问题
### CMD、ENTRYPOINT
`CMD`是容器启动时默认执行命令；
`ENTRYPOINT`是镜像的默认入口命令；
每个`Dockerfile`两者都只能有一个，后面的会代替前面的，`CMD`可以是提供给`ENTRYPOINT`的默认参数。

## .`dockerignore`
+ `*`表示任意多字符；
+ `?`表示单个字符；
+ `!`表示不匹配。

例如：
```
logs
!logs/.gitkeep
*.pyc
tmp?
```

## 示例
```
FROM python:3.7

WORKDIR /opt/proj
COPY requirements.txt /opt/proj
RUN pip install -r requirements.txt -i http://mirrors.aliyun.com/pypi/simple/ --trusted-host mirrors.aliyun.com

COPY . /opt/proj

ENTRYPOINT ["/bin/bash"]
```

# docker-compose
[docker-compose](https://docs.docker.com/compose/compose-file/)

## 安装
`docker-compose`是基于`Python`开发，通过`pip`安装
```
pip3 install docker-compose
```

##  示例
通过`docker-compose`本地数据卷的方式启动`Django Web`服务、`Celery worker`和`Celery beat`。
```
version: "3.7"
services:
  # REST API
  rest:
    build: .
    image: proj
    volumes:
      - .:/opt/proj
    expose:
      - 8000
    entrypoint:
      - daphne
      - -b
      - 0.0.0.0
      - proj.asgi:application
    ports:
      - "8000:8000"
  # Celery worker
  worker:
    build: .
    image: proj
    volumes:
      - .:/opt/proj
    entrypoint:
      - celery
      - -A
      - proj
      - worker
      - -l
      - info
  # Celery beat
  beat:
    build: .
    image: proj
    volumes:
      - .:/opt/proj
    entrypoint:
      - celery
      - -A
      - proj
      - beat
      - -l
      - info
```
