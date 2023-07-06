---
title: apache deploy django
date: 2023-07-03 17:29:20
tags:
---

## 安装apache模块mod_wsgi

    编译安装会指定参数--with-apxs,执行 `sudo apt-get install apache2-dev`

1. 下载源码包
mod_wsgi的源码托管在Github上，你可以从https://github.com/GrahamDumpleton/mod_wsgi/releases下载它各个版本的源码包。

2. 解压`tar -xzvf mod_wsgi-4.9.4.tar.gz`，配置编译选项，指定apache和python环境

    sudo ./configure --with-apxs=/usr/bin/apxs --with-python=/usr/local/python3/bin/python3
3. 编译并安装
    sudo make && sudo make install

4. 查看生成的.so文件
    ll /usr/lib/apache2/modules/mod_wsgi.so
5. 激活wsgi模块
    /etc/apache2/mods-enabled/ 已经被启用的模块
    /etc/apache2/mods-available/ 当前系统中可用的模块
    a2enmod 模块名 ---启用某模块
    a2dismod 模块名 ----禁用某模块

    a2enmod wsgi

## 编写网站配置文件


1. cd /etc/apache2/sites-available

2. sudo vi py.conf
```apache
<VirtualHost *:8888>
   ServerName localhost
   WSGIScriptReloading On
   WSGIScriptAlias / /home/ehigh/work/html/navigation_site/navigation_site/wsgi.py
   WSGIDaemonProcess jango-web python-home=/home/ehigh/work/html/navigation_site/venv python-path=/home/ehigh/work/html/navigation_site processes=2 threads=15 display-name=%{GROUP}
   WSGIProcessGroup jango-web
   <Files wsgi.py>
        Require all granted
   </Files>
   #错误日志
   ErrorLog ${APACHE_LOG_DIR}/django-myproject-error.log
   CustomLog ${APACHE_LOG_DIR}/myproject-django.log combined
</VirtualHost>
```

3. cd /etc/apache2/sites-enabled && sudo ln -s ../sites-available/py.conf py.conf
## 创建django项目的虚拟环境
1. cd /home/ehigh/work/html/navigation_site

2. /usr/local/python3/bin/python3 -m venv venvpython3 -m venv venv && source venv/bin/activate

3. pip install -r requirements.txt

4. vi navigation_site/wsgi.py追加以下代码
```python
# 追加navigation_site搜索路径
import os
path = os.path.dirname(os.path.dirname(os.path.abspath(__file__)))
if path not in sys.path:
    sys.path.append(path)
```

## 重启apache

1. systemctl restart apache2

2. curl 192.168.12.234:8888

<!--more-->
