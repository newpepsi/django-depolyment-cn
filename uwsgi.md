# 在ubuntu中部署使用uWSGI服务部署Django
## 参数说明
>ubuntu版本：14.04
>
>django项目路径 /var/www/djangoproject/
>
>网址： www.mydomain.com


## 1. 安装nginx服务器
### ubuntu下安装nginx
    sudo apt-get install nginx


## 2. 配置nginx服务器

### 2.1 配置虚拟站点
    cd /etc/nginx/sites-available/
    touch uwsgisite
    vi uwsgisite
    
### 2.2 编辑虚拟站点配置内容
    
    server{
        listen 80;
        # 访问你django项目所需要的域名 多个域名可以用【空格】分开
        server_name www.mydomain.com w1.mydomain.com;
        #django 静态文件映射
        location /static/ {
            #指向我们django目录中的static文件夹
            root /var/www/djangoproject/static/;
        }
        
        #django映射
        location / {
            include uwsgi_params;
            uwsgi_pass 127.0.0.1:3031;
            
            #如果django中使用上传功能此处需要设置最大上传尺寸
            client_max_body_size 50m; 
        }
    }
 保存文件后进行如下操作,我们将uwsgisite 文件进行软连接
 并重新启动nginx服务器

    sudo ln -s uwsgisite /etc/nginx/sites-enabled/uwsgisite
    sudo service nginx reload

## 3. 使用uwsgi部署django
### 3.1 安装 uwsgi
首先安装依赖库，并把pip升级到最新版本

    sudo apt-get install python-dev
    sudo apt-get install python-pip
    sudo pip install pip --upgrade
    sudo apt-get install libpcre3 libpcre3-dev
    sudo apt-get install zlib1g-dev
    sudo apt-get install nginx-full
为保证后续正常工作我们先卸载旧版的uwsgi：

    sudo pip uninstall uwsgi
    sudo apt-get remove uwsgi
重新安装uWSGI

    sudo pip install uwsgi
    
### 3.2 编写uWSGI配置文件
    cd /var/www/djangoproject/
    touch djangoproject/uwsgi.ini
    vi djangoproject/uwsgi.ini
在 /var/www/djangoproject/djangoproject/uwsgi.ini 文件中插入

    [uwsgi]
    # uwsgi监听以socket模式监听本地 3031端口
    socket = 127.0.0.1:3031
    
    # django项目的根目录
    chdir = /var/www/djangoproject/
    
    # django项目中 wsgi.py 文件所在地址
    wsgi-file = djangoproject/wsgi.py
    
    #工作进程数量
    processes = 4
    threads = 2
    stats = 127.0.0.1:9191


### 3.3 使用uWSGI启动django项目
    cd /var/www/djangoproject/
    uwsgi djangoproject/uwsgi.ini
    
在 /var/www/djangoproject/djangoproject/uwsgi.ini 文件末尾加入

    # 让uWSGI工作在后台模式
    daemonize = /var/www/djangoproject/uwsgi.log


### 3.4 将uwsgi加入到开机启动文件
    cd /var/www/djangoproject
    
    sudo touch djangosite.sh
    sudo vi djangosite.sh
    
插入内容

    #!/bin/sh
    cd /var/www/djangoproject
    /usr/bin/uwsgi /var/www/djangoproject/djangoproject/uwsgi.ini
    
保存 djangosite.sh 文件后

    cd /var/www/djangoproject
    chmod a+x djangosite.sh
    sudo vi /etc/rc.local
    
在打开的 /etc/rc.local 文件中插入
    
    /var/www/djangoproject/djangosite.sh
    exit 0

保存以后，下次启动系统的时候我们uwsgi就会自动启动。
    
> 受篇幅限制我们没有讲解如何使用系统服务的方式来启动uwsgi。
> 将程序加入到系统服务过程复杂，我们将会单独讲解如何将程序加入系统服务


## 4 查看执行结果
在浏览器中输入
    
    http://www.mydomain.com
    


1.  [查看uWSGI 部署django官方文档](http://uwsgi-docs.readthedocs.io/en/latest/tutorials/Django_and_nginx.html)

**我的其它相关文章**

-   [基于gunicorn部署django](http://note.youdao.com/noteshare?id=623eb7a352631845afc61de7a92c9d26&sub=1BF18AE2C05D4FC3B79FFC5AA9340BA7)
-   

**QQ讨论群**

Django中国：122241953

群内有热心管理指导大家学习