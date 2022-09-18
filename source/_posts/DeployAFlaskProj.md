---
title: Deploying Flask project on CentOS 8
tags:
  - Python
  - Linux
categories: Linux & Shell
date: 2021-07-03 12:06:54
---



# Preface
The reason for writing this note is I met a lot of problems that do not have direct solutions that can be found online. I thought writing it down could help a lot as I may still face similar issues in the future.
<!--more-->

# About the server

I purchased a server provided by Ali cloud, which was on a almost 60% discount when I bought it. 

# About dependencies

## Python version

***yum*** is a convenient instruction on CentOS. ***yum*** provides 4 versions of ***Python***:

- python2 
- python36
- python38
- python39
 
For instance, if you wish to install python 3.9, simply type in
```
yum install python39
```
If you are prompted with permission denied, add ```sudo``` before the command.

## Virtual environment

Why is a venv (vertual environment) needed?

The good thing about venv is that it has a scope limited to one project. It will not affect your system-level settings, and it is independent from other projects. For instance, when one of your project needs Fabric 1.x and another needs Fabric 2.x, and both project has venv, there will not be a conflict. However, if you do not have venv enabled, you will have to delete one version of Fabric.

The official documentary suggests using
```
python3 -m venv (/path to new virtual environment)
```
to create a new venv.

For instance, you can use
```
python3 -m venv venv
```
to create a venv under current working directory.

After that, use
```
source ./venv/bin/activate
```
to activate venv.

To deactivate venv, simply type in
```
deactivate
```
.

## Dependencies

This part requires your ***Flask*** project to be all set, waiting to be synchronized to the server. 

You can use
```
pip freeze > requirements.txt
```
to extract all packages installed in the current venv, and save them into a file named 'requirements.txt'.

To install all dependencies on server, simply type in:
```
pip install -r requiremtns.txt
```
Note that if you are prompted permission denied, Python documentary recommends using '--user' option instead of adding a sudo. The effect of this option is to install the package into current user's home directory, instead of under where python is installed.

# Install ***Nginx***

***Nginx*** is already packed by ***yum***, so we can use

```
sudo yum install nginx
```
to install nginx.

After installation, you can check if the installation was successful by
```
nginx -v
```
.

These are a few common commands:
```
sudo systemctl enable nginx //Start Nginx upon booting
sudo systemctl start nginx //Start Nginx
sudo systemctl stop nginx //Shut down Nginx
sudo systemctl restart nginx //Reload Nginx
sudo systemctl status nginx //Check status of Nginx
```

# Install ***uwsgi***

You can install ***uwsgi*** through
```
pip install uwsgi
```
.

After installation, you can check if the installation was successful by
```
uwsgi --version
```
.

# Deploy to server

Connect to your server, enter the directory under which you want your project to be placed.

Synchroze your project with git, scp or sftp.

Make sure the python version on your server matches that on your computer.

Then, you can proceed to install dependencies using
```
pip install -r requirements.txt
```	
.

## ***uwsgi*** initialization

Create a file named 'uwsgi.ini' under your project's root directory.
In the file, type in
```
[uwsgi]
socket=127.0.0.1:5000 //The port can be altered.

chdir=/home/wst/wms //Your project's root directory.

wsgi-file=app.py //Your start file. Note this is a relative path to your root dir.

callable=app //The name of your start file instance.

processes=4 

threads=2

master=true

stats=127.0.0.1:9191

```
Remember to delete the side notes. ***uwsgi*** cannot tell the difference between comments and scripts.

## Nginx configuration

The configuration file for ***Nginx***, 'nginx.conf', is usually located under '/etc/nginx' directory. If you cannot find it, type in
```
sudo nginx -t
```
to see where is it.

Open it with a text editor:
```
sudo vim /etc/nginx/nginx.conf
```
, and you will see something like:
```
# For more information on configuration, see:
#   * Official English Documentation: http://nginx.org/en/docs/
#   * Official Russian Documentation: http://nginx.org/ru/docs/

user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log;
pid /run/nginx.pid;

# Load dynamic modules. See /usr/share/doc/nginx/README.dynamic.
include /usr/share/nginx/modules/*.conf;

events {
    worker_connections 1024;
}

http {
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile            on;
    tcp_nopush          on;
    tcp_nodelay         on;
    keepalive_timeout   65;
    types_hash_max_size 2048;

    include             /etc/nginx/mime.types;
    default_type        application/octet-stream;

    # Load modular configuration files from the /etc/nginx/conf.d directory.
    # See http://nginx.org/en/docs/ngx_core_module.html#include
    # for more information.
    include /etc/nginx/conf.d/*.conf;

    server {
        listen       80 default_server;
        listen       [::]:80 default_server;
        server_name  _;
        root         /usr/share/nginx/html;

        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;

        location / {
        }

        error_page 404 /404.html;
            location = /40x.html {
        }

        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }
    }

# Settings for a TLS enabled server.
#
#    server {
#        listen       443 ssl http2 default_server;
#        listen       [::]:443 ssl http2 default_server;
#        server_name  _;
#        root         /usr/share/nginx/html;
#
#        ssl_certificate "/etc/pki/nginx/server.crt";
#        ssl_certificate_key "/etc/pki/nginx/private/server.key";
#        ssl_session_cache shared:SSL:1m;
#        ssl_session_timeout  10m;
#        ssl_ciphers PROFILE=SYSTEM;
#        ssl_prefer_server_ciphers on;
#
#        # Load configuration files for the default server block.
#        include /etc/nginx/default.d/*.conf;
#
#        location / {
#        }
#
#        error_page 404 /404.html;
#            location = /40x.html {
#        }
#
#        error_page 500 502 503 504 /50x.html;
#            location = /50x.html {
#        }
#    }

}
```
Change 'server_name' inside 'server' to your ip address or your domain name.

Add two lines under 'root'. One is 'access_log', the other is 'error_log'.

Enable 'uwsgi_params' in 'location /'.

```
    server {
        listen       80 default_server;
        listen       [::]:80 default_server;
        server_name  xxx.xxx.xxx.xxx;
        root         /usr/share/nginx/html;
        access_log  /home/wst/wms/logs/access.log;
        error_log  /home/wst/wms/logs/error.log;

        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;

        location / {
       		include     uwsgi_params;
        	uwsgi_pass      127.0.0.1:5000; #这里与uwsgi配置文件里的socket保持一致
	        uwsgi_param UWSGI_PYHOME /home/wst/wms/venv; #python虚拟环境的地址
	        uwsgi_param UWSGI_CHDIR  /home/wst/wms; #项目根目录地址
	        uwsgi_param UWSGI_SCRIPT app:app; #启动脚本名:实例名
        }

        error_page 404 /404.html;
            location = /40x.html {
        }

        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }
    }
```
Then use
```
sudo nginx -t
```
to check if there is any grammatical error.
If you are not prompted an error, type in
```
sudo systemctl restart nginx
```
.
Now you should be able to see your web app running on your domain or your ip address.
