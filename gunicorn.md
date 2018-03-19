# 在ubuntu中使用gunicron部署Django
## 参数说明
ubuntu版本：14.04

django项目路径 /var/www/djangoproject/

网址： www.mydomain.com

python 版本2.7


## 1. 安装nginx服务器
### ubuntu安装
    sudo apt-get install nginx


## 2. 配置nginx服务器

### 2.1 配置虚拟站点
    cd /etc/nginx/sites-available/
    touch djangosite
    vi djangosite
    
### 2.2 编辑虚拟站点配置内容
    
    server{
        listen 80;
        server_name www.mydomain.com;
        #django 静态文件映射
        location /static/ {
            #指向我们django目录中的static文件夹
            alias /var/www/djangoproject/static/;
        }
        
        #django映射
        location / {
            proxy_pass http://127.0.0.1:8000;
            include proxy_params;
            
            #如果django中使用上传功能此处需要设置最大上传尺寸
            client_max_body_size 50m; 
        }
    }
 保存文件后进行如下操作

    sudo ln -s djangosite /etc/nginx/sites-enabled/djangoproject
    sudo service nginx reload

## 3 安装gunicorn

    sudo pip install gunicorn

### 3.1 使用gunicorn 执行django项目
    cd /var/www/djangoproject/
    gunicorn djangoproject.wsgi -b127.0.0.1:8000 -w 3
    
#### 我们来介绍下上面例子中使用gunicorn的各个参数

1. djangoproject.wsgi 这个是我们的项目目录同时也是我们这个项目应用名称
 使用 ==djangoproject.wsgi== 指定启动 djangoproject 这个项目
1. ==-b127.0.0.1:8000== 参数是将我们gunicorn 绑定到本机 某个IP及端口上
1. ==-w 3== 参数表示我们将使用3个同步的工作线程来提供请求处理服务

[查看gunicorn参数文档](http://docs.gunicorn.org/en/latest/run.html#commonly-used-arguments)

### 3.2 使用gunicorn 小技巧
通常情况下我们不可能每次都这么去输入长串的代码来启动gunicorn服务
所以我们写个sh脚本来执行连续的命令

创建我们启动gunicorn的脚本
    
    touch production.sh
    
在脚本中插入
    
    #!/bin/sh
    cd /var/www/djangoproject
    nohup gunicorn djangoproject.wsgi -b127.0.0.1:8000 -w 3 &
    
    ##保存退出
    
接着执行，修改脚本的执行权限
    
    chmod a+x production.sh
    
启动gunicorn
    
    ./production.sh
    
    

## 4 查看执行结果
在浏览器中输入
    
    http://127.0.0.1:8000
    

**我的其它相关文章**

-   [基于uWSGI部署django](https://github.com/newpepsi/django-depolyment-cn/blob/master/uwsgi.md)

**QQ讨论群**

Django中国：122241953

群内有热心管理指导大家学习