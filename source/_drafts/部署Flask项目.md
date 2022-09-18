---
title: 在CentOS 8上部署基于Flask实现的项目
date: 2021-07-03 12:06:54
categories: Linux & Shell
tags:
- Python
- Linux
---


# 写在前面
写这篇文章是因为我在自己部署项目的时候，遇到了很多仅靠别人的博客和回答解决不了的问题。于是在自己解决了之后，就希望能够详细的记录下来，既能方便其他可能跟我一样基础比较差的人，也预防将来的自己忘掉了，又得一遍遍的翻枯燥的官方文档。
<!--more-->
# 关于服务器
我购买的是阿里云的服务器，今年618期间很便宜，最低配置的60块不到就能买上一年。
操作系统我选择了CentOS 8。原因无他，一直以来用的都是Cent OS，且更高的版本号意味着更好的兼容性与稳定性（测试版除外）。
# 关于环境
以下这些操作，建议在本地电脑完成之后，再使用git同步到服务器端。

非常不建议用户直接使用root账户来进行以下操作。

非常建议用户创建一个有sudo权限的账户来进行以下操作。
## python版本
在Cent OS下，建议直接使用yum命令进行下载。yum命令提供了4个版本的python可供下载:

 - python2 
 - python36
 - python38
 - python39
 
 例如下载python 3.9时，直接使用命令
 ```
 yum install python39
 ```
 如果提示权限不够记得加上sudo。
 
 版本可以根据个人使用习惯来进行选择。我个人一直使用的是python 3.8。

不推荐使用python2，尽管我看到的很多教程都说python 2与Flask和uWSGI相性更佳，但实际使用下来python3也完全没有出现不兼容的问题。

## 虚拟环境
为什么要创建虚拟环境？

虚拟环境的好处在于安装在其中的插件不会影响到你电脑上的本地配置，也独立于其他的虚拟环境。例如当你的一个项目需要Fabric 1.x，而另一个需要Fabric 2.x的时候，如果你使用了虚拟环境，他们互相不会冲突。但如果不使用虚拟环境，你就需要删掉其中一个版本，再安装另一个版本以避免兼容性问题。

在python 3中，官方文档建议用户使用
```
python3 -m venv (/path to new virtual environment)
```
命令来创建虚拟环境。

例如，你可以使用
```
python3 -m venv venv
```
来在当前目录下创建一个目录名为venv的虚拟环境。

创建完成之后，进入到venv文件，使用
```
source ./bin/activate
```
来激活虚拟环境，处在虚拟环境中时，直接输入
```
deactivate
```
来退出虚拟环境。

## 依赖包
这一步需要等你的Flask项目已经万事俱备，只差部署到服务器端的时候再操作，写在这里只是因为与上边的一块内容有些许关联。

用
```
pip freeze > requirements.txt
```
命令可以提取当前所有依赖包，并将其写入requirements.txt文件中，这样在服务器端安装时，可以直接使用
```
pip install -r requiremtns.txt
```
安装你在本地电脑上的所有依赖包。
需要注意的是如果提示权限不够，pip的官方文档并不提倡使用sudo，而是建议用户使用“ --user ”参数。这个参数的含义是将包安装到用户主目录，而非python的安装目录下。
# 安装nginx
由于nginx在近几年变得越来越流行，现在yum的库里已经包含了nginx，不再需要我们手动添加了。

直接使用
```
sudo yum install nginx
```
来安装。

安装完可以使用
```
nginx -v
```
命令来检查是否安装成功。

以下是几个常用命令：
```
sudo systemctl enable nginx //开机启动nginx
sudo systemctl start nginx //启动nginx
sudo systemctl stop nginx //关闭nginx
sudo systemctl restart nginx //重新加载nginx
sudo systemctl status nginx //查看nginx状态
```
关于nginx的配置问题会在最后讲解。

# 安装uwsgi
关于uwsgi的作用以及他与nginx的区别，想要了解的可以参考这篇文章：
[nginx和uwsgi的区别和作用](https://www.dazhuanlan.com/2019/10/11/5d9f74310baf0/).
文章讲的很好，也很详细，我就不在此赘述了。

安装uwsgi可以使用
```
pip install uwsgi
```
命令直接进行安装。

安装完之后可以使用
```
uwsgi --version
```
命令来确认是否安装成功。

uwsgi的配置问题也会在最后讲解。

# 部署到服务器
连接到你的服务器，进入到你想要放置你项目位置的文件夹。

如果对此犹豫不决，我建议你就放在你当前用户的主目录下。

按个人喜好，使用git或者scp或者ftp传输工具将项目上传到服务器上。

如果你的服务器上没有跟你本地版本相同的python，你需要在服务器端再次重复上述安装python的步骤。

在确保python版本一致之后，你可以使用上边的
```
pip install -r requirements.txt
```	
来安装所有依赖包了。


## uwsgi初始化文件
在你项目的根目录创建一个.ini文件，名称最好能体现他的作用，例如uwsgi.ini或项目名.ini。
在里边写上如下内容：
```
[uwsgi]
socket=127.0.0.1:5000 //你可以选择不同的端口，下边还会用到。

chdir=/home/wst/wms //你的项目的根目录

wsgi-file=app.py //你的启动文件，注意这里是相对项目根目录的地址。

callable=app //启动文件实例的名称

processes=4 

threads=2

master=true

stats=127.0.0.1:9191

```
以上标有注释的请根据实际情况修改，无注释的，也没有需求理解这些参数的可以照抄。

修改完后记得删掉我的注释。

## nginx配置
nginx的配置文件" nginx.conf "通常存在于" /etc/nginx "目录下，如果你找不到，可以使用
```
sudo nginx -t
```
命令来查看他的位置。

使用文本编辑器打开它
```
sudo vim /etc/nginx/nginx.conf
```
会看到类似下面的代码
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
将" server "内的" server_name "改成你的域名或者公网地址；

在" root "下增加两行，分别为"access_log"和"error_log"；

在" location / "内启用" uwsgi_params "。

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
然后使用
```
sudo nginx -t
```
命令检查是否有语法错误。

如果没有报错，输入
```
sudo systemctl restart nginx
```
在此之后，访问你的公网地址，你应该就能看到你的网页了。

下一步就是在服务器端启用监管以重启挂掉的程序，和使用fabric或jenkins来进行持续集成，避免每次在本地更新代码后，还需要进行冗余的操作来将其同步到云端。这部分内容我会在之后完成相应的工作后也写一篇博客记录。

