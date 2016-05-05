## 阿里云配置django环境

> 配置环境：CentOS 6.5 + python 2.7.9 + django 1.8.2

#### 1. 安装python2.7.9

* 安装devtoolset

```linux
yum groupinstall "Development tools"
```

* 下载并解压Python 2.7.9的源代码

```
wget --no-check-certificate https://www.python.org/ftp/python/2.7.9/Python-2.7.9.tar.xz
tar xf Python-2.7.9.tar.xz
```

* 编译与安装Python 2.7.9

```
./configure --prefix=/usr/local
make && make altinstall
```

* 将python命令指向Python 2.7.9

```
ln -s /usr/local/bin/python2.7 /usr/local/bin/python
```

#### 2. 安装pip

```
wget --no-check-certificate https://bootstrap.pypa.io/get-pip.py
python get-pip.py
```

#### 3. 安装virtualenv

```
yum install python-virtualenv
```

#### 5. 安装django
```
pip install django==1.8.2
```

#### 6. 安装python-mysql支持
```
pip install MySQL-python
```

#### 7. 创建项目目录
```
mkdir /alidata/www
mkdir /alidata/www/example_com
cd /alidata/www/example_com
mkdir venv conf src logs
```

#### 8. 创建python虚拟环境
```
virtualenv /alidata/www/example_com/venv
source /alidata/www/example_com/venv/bin/activate
```

#### 9. 创建django项目
```
cd /alidata/www/example.com/src
django-admin startproject yourproject
```

#### 10. 创建uwsgi.ini文件
```
vi /var/www/example_com/conf/uwsgi.ini
```

> uwsgi.ini文件内容

```
[uwsgi]
chdir = /alidata/www/example_com/src/yourproject
module = yourproject.wsgi
home = /alidata/www/example_com/venv

socket = 127.0.0.1:8001
master = true
processes = 10
vacuum = true
logto = /alidata/www/example_com/logs/uwsgi.log
daemonize = /alidata/log/uwsgi/yourproject.log
```

#### 11. 创建nginx配置文件
```
vi /alidata/server/nginx/conf/vhosts/example_com.conf
```

> nginx配置文件内容

```
server {
    listen 80;
    server_name example.com;

    access_log /alidata/www/example_com/logs/access.log;
    error_log /alidata/www/example_com/logs/error.log;

    location /static/ { # STATIC_URL
        alias /alidata/www/example_com/src/static/; # STATIC_ROOT
        expires 30d;
    }

    location /media/ { # MEDIA_URL
        alias /alidata/www/example_com/src/media/; # MEDIA_ROOT
        expires 30d;
    }

    location / {
        include /alidata/server/nginx/conf/uwsgi_params;
        uwsgi_pass 127.0.0.1:8001;
    }
}
```

#### 12. 设置django配置文件
```
vi /alidata/www/example_com/src/yourproject/yourproject/setting.py
```

> 设置数据库

```
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'yourdatabasename',
	'USER': 'root',
	'PASSWORD': 'password',
	'HOST': '127.0.0.1',
	'PORT': '3306',
    }
}
```
> 设置静态目录

```
STATIC_ROOT = '/alidata/www/example_com/src/static/'
```

#### 13. 初始化数据库，同步静态文件到nginx设置的目录下

```
cd /alidata/www/example_com/src/yourproject/
python manage.py migrate
python manage.py collectstatic
```

#### 14. 设置uwsgi服务开机启动

* 软连接uwsgi.ini文件

```
mkdir /etc/uwsgi
mkdir /etc/uwsgi/vassals
ln -s /var/www/example_com/conf/uwsgi.ini /etc/uwsgi/vassals/
```

* 设置开机启动

```
vi /etc/rc.local
```
> 文件中添加

```
/usr/local/bin/uwsgi --master --emperor /etc/uwsgi/vassals --uid www --gid www
```
