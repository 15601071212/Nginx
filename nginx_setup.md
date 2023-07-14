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

首先确认Django项目网站可以通过自带的开发服务器正常运行如下所示：

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

- 通过浏览器可以正常访问Django项目网站地址 http://10.229.191.63:8888/admin/ 如下所示：

![图片](hello_world.jpg)

接下来可以通过uWSGI来运行Django项目网站如下所示：

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

- 通过浏览器可以正常访问Django项目网站地址 http://10.229.191.63:8888/admin/ 如下所示：

![图片](hello_world.jpg)
