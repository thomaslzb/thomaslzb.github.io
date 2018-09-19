---
layout:     post
title:      Python Flask 框架终极部署
subtitle:   Uwsgi+Nginx+mysql+Supervisor+Virtualenv
date:       2018-09-19
author:     LZB
header-img: img/post-bg-hacker.jpg
catalog: true
tags:
    - Python
---

# Python Flask 框架终极部署 #
## Uwsgi+Nginx+mysql+Supervisor+Virtualenv, 基于阿里云默认Linux ##

### 处理本地的代码 ###
假设你已经完成了项目的开发，本地已经安装了git，那么首先将你的代码提交到git;

	#进项目根目录
	pip freeze > requirements.txt  #导flask 全部包，方便新环境下一次性安装。
	git init # 之前如果没有做，那么需要做
	git add --all #提交所有修改后的文件
	git remote add origin http:xxxx.git  #这一步如果之前没做，那么你需要做。
	git commmit -w 'first commit'


### 安装Mysql ###

- 安装mysql, 注意一定记住root 密码，还要在配置文件中设置字符为UTF8！
- 增加新的数据库，注意名字和你项目的一样，create database xxx;


### 安装虚拟环境 Vitualenv ###

- 直接安装虚拟环境容器来顺便安装virtualenv

		pip install virtualenvwrapper
 
- 注意需要先修改环境变量。否则命名无法使用

		vim ~/.bashrc   在最底部增加

- 这是为了让 WORKON 命令能用。

	    export WORKON_HOME=$HOME/.virtualenvs
	    source /usr/local/bin/virtualenvwrapper.sh
        

- 然后你需要用 mkvirtualenv xxx   来创建一个叫xxx 的python 虚拟环境

	workon xxx #切换到你的虚拟环境

### 上传并部署源码 ###
现在在服务器上安装git 拿到源码

	apt install git
	git init
	git remote add origin http:xxx.git
	git pull origin master # 拿到全部源码

部署源码

	进去项目根目录：
	pip install -r requirements.txt  #导入你项目的flask 依赖
	pip manager.py db migerate #初始化数据库  
	# 这里如果出现初始化失败，那么清空你的 migration 文件夹，你会丢掉你的测试数据
	pip manager.py db upgrade #导入数据库，代码迁移完成

### 安装Nginx ###

	apt install nginx #安装nginx 
	贴下配置文件位于： ／etc/nginx/conf.d/xxx.conf
	# 配置nginx与uwsgi的通信方式和名称，mysite就是名称
	upstream xxx {
	    # nginx使用socket的方式与uwsgi进行通信
	    # 这里指向你项目的目录下的socket文件，
	    # 这个socket文件不需要单独创建，在运行的时候会自动创建。
	    server unix:///srv/xxx/xxx.sock;
	}
	# 配置服务器
	server {
	    # 监听的端口号
	    listen 80;
	    # 域名或者本机的ip地址
	    server_name dev.wudizu.com 47.104.22.138;
	    charset     utf-8;
	    # 最大的上传大小
	    client_max_body_size 75M;  
	    # adjust to taste
	
	    # 访问根目录下的任何url的配置
	    location / {
	        # 指定访问哪个upstream
	        uwsgi_pass wudi;
	        # 包括uwsgi参数的文件
	        include     /etc/nginx/uwsgi_params;
	    }
	    location /static {
	        alias /srv/xxx/static;
	   }
	
	}

### 安装Uwsgi ###
	
	pip install uwsgi
	
	贴下配置文件：
	
	
	[uwsgi]
	chdir           = /srv/xxx
	# 模块
	module          = firstweb   #注意这里一定要你写的flask 首页文件名
	# python的虚拟环境
	home            = /root/.virtualenvs/flask-py2
	
	# 是否启用mater模式
	master          = true
	
	# 进程数
	processes       = 10
	
	# socket文件地址
	
	socket          = /srv/xxx/xxx.sock
	
	# wsgi文件
	
	wsgi-file       = /srv/xxx/xxx.ini
	
	# wsgi文件中的app变量
	
	callable        = app
	
	# socket文件的权限
	
	chmod-socket    = 666


### 安装Supervisor ###

	sudo pip install supervisor
	# supervisor的程序名字
	[program:mysite]
	# supervisor执行的命令
	command=uwsgi --ini mysite_uwsgi.ini
	# 项目的目录
	directory = /srv/xxx 
	# 开始的时候等待多少秒
	startsecs=0
	# 停止的时候等待多少秒
	stopwaitsecs=0# 自动开始
	autostart=true
	# 程序挂了后自动重启
	autorestart=true
	# 输出的log文件
	stdout_logfile=/srv/xxx/log/supervisord.log
	# 输出的错误文件
	stderr_logfile=/srv/xxx/log/supervisord.err
	
	[supervisord]
	# log的级别
	loglevel=info
	
	# 使用supervisorctl的配置
	[supervisorctl]
	# 使用supervisorctl登录的地址和端口号
	serverurl = http://127.0.0.1:9001
	
	# 登录supervisorctl的用户名和密码
	username = admin
	password = 123
	
	[inet_http_server]
	# supervisor的服务器
	port = :9001
	# 用户名和密码
	username = admin
	password = 123
	
	[rpcinterface:supervisor]
	supervisor.rpcinterface_factory = supervisor.rpcinterface:make_main_rpcinterface
	
	
	然后使用supervisor运行uwsgi：
	supervisord -c supervisor.conf
	
	使用supervisorctl管理supervisord：supervisor是一个C/S模型，跟redis一样，有一个服务器，也有一个客户端命令用来管理服务的，使用以下命令进入到命令界面：
	supervisorctl -c supervisor.conf
	
	指定的文件必须和supervisord服务端保持一致。 一些常用的命令有：
	supervisorctl -c supervisor.conf
	> status    # 查看状态
	> start program_name # 启动程序
	> restart program_name # 重新启动程序
	> stop program_name # 停止程序
	> reload # 重新加载配置文件
	> quit # 退出当前的客户端


# END #