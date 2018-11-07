---
title: flask部署
date: 2016-11-5 19:08:13
tags: [flask]
---

# uwsgi配置

保存为`config.ini`

```
[uwsgi]

socket = 127.0.0.1:8001

chdir = /home/ubuntu/myblog

wsgi-file = manage.py

callable = app

processes = 4

threads = 2

stats = 127.0.0.1:9191
```

启动：`uwsgi config.ini`

配置文件`/etc/supervisor/conf.d/myblog_supervisor.conf`

# supervisor配置

```
[program:myblog]

command=uwsgi /home/ubuntu/myblog/config.ini

directory=/home/ubuntu/myblog

user=ubuntu

autostart=true
autorestart=true

stdout_logfile=/home/ubuntu/myblog/logs/uwsgi_supervisor.log
```

启动：`service supervisor  start`


# nginx配置

配置文件`/etc/nginx/sites-enabled/default`

```
server{
        listen 80;
        server_name xxx.xxx.xxx.xxx;

        location / {
                include uwsgi_params;
                uwsgi_pass 127.0.0.1:8001;
                uwsgi_param UWSGI_PYHOME /home/ubuntu/myblog/venv;
                uwsgi_param UWSGI_CHDIR /home/ubuntu/myblog;
                uwsgi_param UWSGI_SCRIPT manage:app;
        }
}

```
