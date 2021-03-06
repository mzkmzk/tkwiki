---
title: "MoinMoin Wiki"
date: 2013-08-17 07:36
---


# MoinMoin Wiki #

[官方网站](http://www.moinmo.in/)

# 安装篇 #

环境: `Nginx` + `uWsgi`

参考官方的[HowTo](http://www.moinmo.in/HowTo)

虽然我用的是Gentoo，但是基本都大同小异，我参考的是 [Ubuntu](http://www.moinmo.in/HowTo/UbuntuQuick) 的安装，非常详细

下面参照官方文档稍微改了一下

## 使用virtualenv构造一个隔离的python环境 ##
官方用了virtualenv，搭建一个隔离的环境

    # Install and activate python virtualenv
    emerge -av  virtualenv
    mkdir -p /var/www/moinmoin
    virtualenv /var/www/moinmoin/pythonenv
    source /var/www/moinmoin/pythonenv/bin/activate

    # Download and install moinmoin:
    cd /tmp
    wget http://static.moinmo.in/files/moin-1.9.6.tar.gz
    tar zxvf moin-1.9.6.tar.gz
    python setup.py install	# 这样就会将一个python包安装在/var/www/moin/pythonenv下

    # Deactivate python virtualenv
    deactivate

    # Copy wiki to /var/www/moinmoin
    cp -r ./wiki /var/www/moinmoin/

    # Copy configs to wiki root directory
    cd /var/www/moinmoin/wiki/
    cp config/wikiconfig.py ./
    cp server/moin.wsgi ./

    # cp moin.wsgi moin_wsgi.py	#TODO


## 做基本的配置 ##

    vim /var/www/moinmoin/wiki/moin.wsgi

    sys.path.insert(0, '/var/www/moinmoin/pythonenv/lib/python2.7/site-packages/')
    sys.path.insert(0, '/var/www/moinmoin/wiki/')

    #Fix permission
    chown nginx:nginx -R /var/www/moinmoin
    chmod o-rwx -R /var/www/moinmoin


## 部署uwsgi ##

    # Add an uwsgi.xml config file:
    vim /var/www/moinmoin/wiki/uwsgi.xml

    <uwsgi>
        <uid>nginx</uid>
        <gid>nginx</gid>
        <plugin>python27</plugin>
        <socket>/tmp/moin.sock</socket>
        <limit-as>256</limit-as>
        <processes>8</processes>
        <memory-report/>
        <vhost/>
        <no-site/>
    </uwsgi>

然后启动uwsgi

    uwsgi -x /var/www/moinmoin/wiki/uwsgi.xml


## 最后配置Nginx ##
贴一下本机的Nginx配置

    # MonMoin
    server {
        listen 10086;
        server_name localhost;

        access_log /var/log/nginx/wiki.access_log main;
        error_log /var/log/nginx/wiki.error_log info;

        location ^~ /wiki {
            include uwsgi_params;
            uwsgi_pass unix:///tmp/wiki.sock;
            uwsgi_param UWSGI_PYHOME /var/www/moinmoin/pythonenv/;
            uwsgi_param UWSGI_CHDIR /var/www/moinmoin/wiki/;
            uwsgi_param UWSGI_SCRIPT moin_wsgi;
            uwsgi_param SCRIPT_NAME /wiki;
            uwsgi_modifier1 30;
        }

        location ^~ /wiki/moin_static196/ {
            alias /var/www/moinmoin/pythonenv/lib/python2.7/site-packages/MoinMoin/web/static/htdocs/;
        }
    }

重启Nginx，打开localhost:10086，基本就OK了

# 使用篇 #

# Read More #

# History #
Create 2013/03/25
