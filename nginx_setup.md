### 1. uWSGI的安装、配置和基础测试

#### a. 通过pip安装uWSGI

`pip install uwsgi`



#### b. 建议到 https://uwsgi-docs.readthedocs.io/en/latest/Download.html 页面，下载Stable/LTS版本的源文件

在ubuntu中，解压源码，然后指定安装位置，将uWSGI 安装好：

- 解压文件

`tar -zxvf uwsgi`

- 进入解压目录

`sudo python3 setup.py install`



#### c. 基础测试

在当前目录下创建test.py文件如下所示：
```python
# test.py

def application(env, start_response):

    start_response('200 OK', [('Content-Type','text/html')])

    return [b"Hello World"] # python3

    #return ["Hello World"] # python2

```
- 运行uWSGI：

`uwsgi --http :8000 --wsgi-file test.py`

- 打开浏览器访问8888端口可以看到如下显示：

![图片](hello_world.jpg)

- 基础测试证明如下所示的访问流程可以正常工作：

`the web client <-> uWSGI <-> Python`

#### d. 通过uWSGI运行Django项目网站代替test.py模块：

- 首先确认Django项目网站可以通过自带的开发服务器正常运行如下所示：

```python
python manage.py runserver 0.0.0.0:8888

root@zdh-web-00:/var/www/LRM# python3 manage.py runserver 0.0.0.0:8888

Watching for file changes with StatReloader

Performing system checks...

System check identified some issues:

WARNINGS:

?: (urls.W005) URL namespace 'admin' isn't unique. You may not be able to reverse all URLs in this namespace

System check identified 1 issue (0 silenced).

January 28, 2023 - 10:46:05

Django version 4.1.5, using settings 'LRM.settings'

Starting development server at http://0.0.0.0:8888/

Quit the server with CONTROL-C.

```

通过浏览器可以正常访问Django项目网站地址 http://10.229.191.63:8888/admin/ 如下所示：

![图片](hello_world.jpg)

- 接下来可以通过uWSGI来运行Django项目网站如下所示：

```python
root@zdh-web-00:/var/www/LRM# uwsgi --http :8888 --module LRM.wsgi

*** Starting uWSGI 2.0.20 (64bit) on [Sat Jan 28 10:51:28 2023] ***

compiled with version: 9.4.0 on 22 January 2023 02:35:16

os: Linux-5.4.0-137-generic #154-Ubuntu SMP Thu Jan 5 17:03:22 UTC 2023

nodename: zdh-web-00

machine: x86_64

clock source: unix

detected number of CPU cores: 8

current working directory: /var/www/LRM

detected binary path: /usr/local/bin/uwsgi

!!! no internal routing support, rebuild with pcre support !!!

uWSGI running as root, you can use --uid/--gid/--chroot options

*** WARNING: you are running uWSGI as root !!! (use the --uid flag) ***

*** WARNING: you are running uWSGI without its master process manager ***

your processes number limit is 31471

your memory page size is 4096 bytes

detected max file descriptor number: 1024

lock engine: pthread robust mutexes

thunder lock: disabled (you can enable it with --thunder-lock)

uWSGI http bound on :8888 fd 4

spawned uWSGI http 1 (pid: 12704)

uwsgi socket 0 bound to TCP address 127.0.0.1:40137 (port auto-assigned) fd 3

uWSGI running as root, you can use --uid/--gid/--chroot options

*** WARNING: you are running uWSGI as root !!! (use the --uid flag) ***

Python version: 3.8.10 (default, Nov 14 2022, 12:59:47)  [GCC 9.4.0]

*** Python threads support is disabled. You can enable it with --enable-threads ***

Python main interpreter initialized at 0x557e98b17980

uWSGI running as root, you can use --uid/--gid/--chroot options

*** WARNING: you are running uWSGI as root !!! (use the --uid flag) ***

your server socket listen backlog is limited to 100 connections

your mercy for graceful operations on workers is 60 seconds

mapped 72904 bytes (71 KB) for 1 cores

*** Operational MODE: single process ***

WSGI app 0 (mountpoint='') ready in 1 seconds on interpreter 0x557e98b17980 pid: 12703 (default app)

uWSGI running as root, you can use --uid/--gid/--chroot options

*** WARNING: you are running uWSGI as root !!! (use the --uid flag) ***

*** uWSGI is running in multiple interpreter mode ***

spawned uWSGI worker 1 (and the only) (pid: 12703, cores: 1)

```

通过浏览器可以正常访问Django项目网站地址 http://10.229.191.63:8888/admin/ 如下所示：

![图片](hello_world.jpg)

上述测试证明如下所示的访问流程可以正常工作：

the web client <-> uWSGI <-> Django



注：Django项目LRM的wsgi.py文件内容如下所示：

root@zdh-web-00:/var/www/LRM/LRM# more wsgi.py

"""

WSGI config for LRM project.

It exposes the WSGI callable as a module-level variable named ``application``.

For more information on this file, see

https://docs.djangoproject.com/en/4.1/howto/deployment/wsgi/

"""

import os

import sys

from django.core.wsgi import get_wsgi_application

from os.path import join,dirname,abspath



PROJECT_DIR = dirname(dirname(abspath(__file__)))

sys.path.insert(0,PROJECT_DIR)



os.environ["DJANGO_SETTINGS_MODULE"] = "LRM.settings"



from django.core.wsgi import get_wsgi_application

application = get_wsgi_application()

os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'LRM.settings')

application = get_wsgi_application()

root@zdh-web-00:/var/www/LRM/LRM#



2. nginx的安装和配置

a. 安装nginx

sudo apt-get install nginx

sudo /etc/init.d/nginx start    # start nginx



b. 配置nginx

在/etc/nginx/sites-available/路径下新建Django LRM项目的配置文件LRM_nginx.conf如下所示：

root@zdh-web-00:/etc/nginx/sites-available# more LRM_nginx.conf

# mysite_nginx.conf

# the upstream component nginx needs to connect to

upstream django {

# server unix:///path/to/your/mysite/mysite.sock; # for a file socket

server 127.0.0.1:8001; # for a web port socket (we'll use this first)

#server unix:/var/www/LRM/LRM.sock;

}

# configuration of the server

server {

# the port your site will be served on

listen      8000;

# the domain name it will serve for

server_name 10.229.191.63; # substitute your machine's IP address or FQDN

charset     utf-8;

# max upload size

client_max_body_size 75M;   # adjust to taste

access_log /var/log/nginx/access.log;

error_log /var/log/nginx/error.log;

# Django media

location /media  {

alias /var/www/LRM/media;  # your Django project's media files - amend as required

}

location /static {

alias /var/www/LRM/static; # your Django project's static files - amend as required

}

# Finally, send all non-media requests to the Django server.

location / {

uwsgi_pass  django;

include     /var/www/LRM/uwsgi_params; # the uwsgi_params file you installed

uwsgi_connect_timeout 30;

}

}

root@zdh-web-00:/etc/nginx/sites-available#

通过ln -s命令建立软连接从 /etc/nginx/sites-enabled到这个文件如下所示：

sudo ln -s /etc/nginx/sites-available/mysite_nginx.conf /etc/nginx/sites-enabled/

root@zdh-web-00:/etc/nginx/sites-enabled# ls -al

total 8

drwxr-xr-x 2 root root 4096 Jan 22 22:31 .

drwxr-xr-x 8 root root 4096 Jan 22 22:24 ..

lrwxrwxrwx 1 root root   34 Jan 22 22:24 default -> /etc/nginx/sites-available/default

lrwxrwxrwx 1 root root   41 Jan 22 22:31 LRM_nginx.conf -> /etc/nginx/sites-available/LRM_nginx.conf

root@zdh-web-00:/etc/nginx/sites-enabled#



注：将Django项目所有的静态文件(static files)放在static文件夹下，并编辑/var/www/LRM/LRM/settings.py文件如下所示

STATICFILES_DIRS = [

os.path.join(BASE_DIR, 'static'),

]



3. 通过uwsgi和nginx运行Django应用

a. 在/var/www/LRM文件夹下新建uwsgi.ini文件如下所示：

root@zdh-web-00:/var/www/LRM# more uwsgi.ini

[uwsgi]

#chdir = /var/www/LRM    //项目根目录

module = LRM.wsgi     //  指定wsgi模块下的application对象

#workers = 5

#pidfile= /var/www/LRM/uwsgi.pid

socket = :8001        //对本机8000端口提供服务

master = true                   //主进程

#vacuum =true

#enable-threads = true

#thunder-lock = true

#harakiri =30

#post-buffer = 4096

#http = 127.0.0.1:8001

#static-map = /static=/var/www/LRM/static

#uid = root

#gid = root

wsgi-file = /var/www/LRM/LRM/wsgi.py

#daemonize = /var/www/LRM/uwsgi.log

logto = /var/www/LRM/uwsgi.log

disable-logging = true   //不记录正常信息，只记录错误信息

#buffer-size =65535

root@zdh-web-00:/var/www/LRM#



b. 通过如下所示的命令停止和启动 nginx + uwsgi：

root@zdh-web-00:~# netstat -lnp| grep 8000

tcp 0 0 0.0.0.0:8000 0.0.0.0:* LISTEN 212910/nginx: maste

root@zdh-web-00:~# netstat -lnp| grep 8001

tcp 0 0 0.0.0.0:8001 0.0.0.0:* LISTEN 212989/uwsgi

root@zdh-web-00:~# kill -9 212910

root@zdh-web-00:~# kill -9 212989

root@zdh-web-00:~# netstat -lnp| grep 8000

root@zdh-web-00:~# netstat -lnp| grep 8001

root@zdh-web-00:~# service nginx start

root@zdh-web-00:~# cd /var/www/LRM

root@zdh-web-00:/var/www/LRM# uwsgi --chdir /var/www/LRM --ini uwsgi.ini

[uWSGI] getting INI configuration from uwsgi.ini

root@zdh-web-00:/var/www/LRM#

root@zdh-web-00:/var/www/LRM#

root@zdh-web-00:/var/www/LRM# netstat -lnp| grep 8001

tcp 0 0 0.0.0.0:8001 0.0.0.0:* LISTEN 213151/uwsgi

root@zdh-web-00:/var/www/LRM# netstat -lnp| grep 8000

tcp 0 0 0.0.0.0:8000 0.0.0.0:* LISTEN 213135/nginx: maste

root@zdh-web-00:/var/www/LRM#

通过浏览器的8000端口可以正常访问Django项目网站地址http://10.229.191.63:8000/admin/如下所示：

![图片](hello_world.jpg)

上述测试证明如下所示的访问流程可以正常工作：

the web client <-> the web server <-> the socket <-> uWSGI <-> Django



附录：参考文档网页地址：

https://uwsgi-docs.readthedocs.io/en/latest/tutorials/Django_and_nginx.html

https://www.liujiangblog.com/course/django/181

