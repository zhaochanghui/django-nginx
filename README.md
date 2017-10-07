

1.安装 uwsgi 及 测试 uwsgi

1.1安装

[root@crazy-acong ~]# pip3 install uwsgi
1.2 测试 uwsgi 提供 web 服务的功能

# 创建 test.py 文件
def application(env, start_response):
    start_response('200 OK', [('Content-Type','text/html')])
    return [b"Hello World"] # python3
    #return ["Hello World"] # python2


# 启动 uwsgi 服务
[root@crazy-acong ~]# uwsgi --http :8000 --wsgi-file test.py 

# 查看启动进程
[root@crazy-acong ~]# netstat -lnpt | grep uwsgi
tcp        0      0 127.0.0.1:26685             0.0.0.0:*                   LISTEN      22120/uwsgi         
tcp        0      0 0.0.0.0:8000                0.0.0.0:*                   LISTEN      22120/uwsgi  

# 在浏览器中访问 http://ip:8000 就可以看到 Hello World 字样了


2.安装测试完毕，正是和nginx搭配教程

项目：/home/admin/django_test
项目名称：django_test
项目根目录：/home/admin

2.1  创建django项目：
[admin@localhost ~]$ django-admin.py  startproject django_test
[admin@localhost ~]$ cd django_test/
[admin@localhost django_test]$ ls
django_test  manage.py
[admin@localhost django_test]$ python3 manage.py  runserver localhost:5555

浏览器访问：localhost:5555 
出现‘It worked!
Congratulations on your first Django-powered page.’创建成功

2.2 创建 uwsgi 配置文件
[admin@localhost django_test]$ pwd
/home/admin/django_test
[admin@localhost django_test]$ vi test-uwsgi.ini
内容如下：

[uwsgi]
# 对外提供 http 服务的端口
http = :5555

#用于和 nginx 进行数据交互的端口,注意,nginx配置要用到这个端口
socket = 127.0.0.1:5556

# the base directory (full path)  django 程序的主目录
chdir = /home/admin/django_test

# Django's wsgi file
wsgi-file = django_test/wsgi.py

# maximum number of worker processes
processes = 4

#thread numbers startched in each worker process
threads = 2
 
#monitor uwsgi status  通过该端口可以监控 uwsgi 的负载情况
stats = 127.0.0.1:5557


# clear environment on exit
vacuum          = true

# 后台运行,并输出日志,确保这个文件存在，注意权限
daemonize = /var/log/uwsgi.log

2.3  通过 uwsgi 配置文件启动 django 程序
[admin@localhost django_test]$ pwd
/home/admin/django_test
[admin@localhost django_test]$ uwsgi test-uwsgi.ini 
[uWSGI] getting INI configuration from test-uwsgi.ini
[admin@localhost django_test]$ ls
db.sqlite3  django_test  manage.py  test-uwsgi.ini

此时，通过 [admin@localhost dj1]$ netstat -tlun
可以看到5555, 5556 , 5557端口都启动了,


3.配置nginx
我的是yum nginx安装的，注释掉nginx.conf中的server部分，
[admin@localhost dj1]$ cd /etc/nginx/conf.d/
新建一个nginx配置文件，比如我的：
[admin@localhost dj1]$ cd /etc/nginx/conf.d/
[admin@localhost conf.d]$ more dj.conf 
# the upstream component nginx needs to connect to
upstream django {
    # server unix:///path/to/your/mysite/mysite.sock; # for a file socket
    server 127.0.0.1:5556; # for a web port socket (we'll use this first)
}
 
# configuration of the server
server {
    # the port your site will be served on
    listen      80;
    # the domain name it will serve for
    server_name  localhost; # substitute your machine's IP address or FQDN
    charset     utf-8;
 
    # max upload size
    client_max_body_size 75M;   # adjust to taste
 


 
  # location /media {
        #静态资源目录
        alias /home/admin/django_test/media; # your Django project's media files - amend as required
   # }

    location /static {
        #静态资源目录
        alias /home/admin/django_test/static; # your Django project's static files - amend as required
    }
 
    # Finally, send all non-media requests to the Django server.
    location / {
        uwsgi_pass  django;
        include    uwsgi_params; # the uwsgi_params file you installed
    }
}
[admin@localhost conf.d]$ 

重启nginx
[admin@localhost conf.d]$ systemctl restart nginx

访问localhost

同样看到'It worked!
Congratulations on your first Django-powered page.'
说明成功了
