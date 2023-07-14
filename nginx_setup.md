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
