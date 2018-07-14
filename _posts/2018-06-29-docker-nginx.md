---
title: Docker 之 Nginx环境搭建
date: 2018-06-29 16:22:00
categories:
- Docker
tags:
- docker
- nginx
---

Docker搭建微服务自动部署 <架构探险之路>，让我们来了解下Docker中如何安装、使用nginx吧！

---

# Docker 之 Nginx环境搭建

## Nginx 安装教程

> Linux 环境

### Linux 中安装

-   安装编译工具及库文件

        yum -y install make zlib zlib-devel gcc-c++ libtool  openssl openssl-devel

-   首先要安装 PCRE

    PCRE 作用是让 Nginx 支持 Rewrite 功能

    > 若提示无wget则安装（在 Docker 拉取的源 centos 中默认是没安装的）

        yum -y install wget

    > 进入程序安装目录

        cd /usr/local/src/

    -   下载pcre安装包

            wget http://downloads.sourceforge.net/project/pcre/pcre/8.35/pcre-8.35.tar.gz

    -   解压安装包:

            tar -zxvf pcre-8.35.tar.gz

    -   删除安装包

            rm -rf pcre-8.35.tar.gz

    -   进入安装包

            cd pcre-8.35

    -   编译安装

            ./configure
            make && make install

    -   查看pcre版本

            preconv --version

        ```linux
        [root@de790c02f7b6 pcre-8.35]# preconv --version
        GNU preconv (groff) version 1.22.2 with iconv support
        ```

-   安装 Nginx

    -   下载安装包

            wget http://nginx.org/download/nginx-1.6.2.tar.gz

    -   安装

             cd nginx-1.8.0
             ./configure --prefix=/usr/local/nginx
             make
             make install

    -   检查安装结果

              /usr/local/nginx/sbin/nginx -v

### Dockerfile 安装

> 脚本

    FROM centos:latest
    ## 制作者信息
    MAINTAINER xiachaoyang xiazhaoyang@live.com
    RUN yum install -y pcre-devel wget net-tools gcc zlib zlib-devel make openssl-devel
    ## 创建目录
    RUN mkdir -p /usr/local/nginx
    ## 本地安装 直接自动解压
    ## ADD nginx-1.8.0.tar.gz /usr/local/src
    ## 在线安装
    RUN cd /usr/local/src
    ADD http://nginx.org/download/nginx-1.8.0.tar.gz .
    RUN tar -zxvf nginx-1.8.0.tar.gz
    RUN ln -s /usr/local/src /componets
    RUN cd nginx-1.8.0 && ./configure --prefix=/usr/local/nginx && make && make install
    RUN rm -vf /usr/local/nginx/conf/nginx.conf
    ADD http://www.apelearn.com/study_v2/.nginx_conf /usr/local/nginx/conf/nginx.conf
    EXPOSE 80
    # 设置启动时执行命令
    #ENTRYPOINT /usr/local/nginx/sbin/nginx -v

> 镜像

  docker build -t env-nginx .
  docker images
  docker tag [镜像ID] env-nginx:1.0
  docker run -i -t -p 80:80 env-nginx:1.0

> 清空 Nginx Docker

  docker rm -f `docker container ls -a -q`
  docker rmi -f env-nginx:2.0
  docker build -t env-nginx:2.0 .
  docker run -i -t -p 8001:80 --name nginx --restart=always env-nginx:2.0

> 运行后执行nginx

    xiazhaoyangdeMacBook-Pro:~ xiazhaoyang$ docker run -i -t -p 8001:80 --restart=always 127.0.0.1:5000/env-nginx:1.0
    [root@56dfba762876 /]# cd usr/local/nginx/sbin/
    [root@56dfba762876 sbin]# ls
    nginx
    [root@56dfba762876 sbin]# ./nginx -v    
    nginx version: nginx/1.8.0
    [root@56dfba762876 sbin]# ps -ef|grep nginx
    root        18     1  0 07:44 pts/0    00:00:00 grep --color=auto nginx
    [root@56dfba762876 sbin]# ./nginx
    [root@56dfba762876 sbin]# ps -ef|grep nginx
    root        20     1  0 07:45 ?        00:00:00 nginx: master process ./nginx
    nobody      21    20  0 07:45 ?        00:00:00 nginx: worker process
    nobody      22    20  0 07:45 ?        00:00:00 nginx: worker process

> 配置路由

-   关闭nginx

    ./nginx -s stop


	    xiazhaoyangdeMacBook-Pro:~ xiazhaoyang$ docker ps
	    CONTAINER ID        IMAGE                          COMMAND                  CREATED             STATUS              PORTS                    NAMES
	    56dfba762876        127.0.0.1:5000/env-nginx:1.0   "/bin/bash"              10 minutes ago      Up 10 minutes       0.0.0.0:8001->80/tcp     brave_kowalevski
	    d75f05b4d7f8        hyper/docker-registry-web      "start.sh"               5 days ago          Up About an hour    0.0.0.0:8000->8080/tcp   local-docker-res-web
	    1b5239bc943b        registry                       "/entrypoint.sh /etc…"   5 days ago          Up About an hour    0.0.0.0:5000->5000/tcp   local-docker-res
	    xiazhaoyangdeMacBook-Pro:~ xiazhaoyang$ docker attach 56dfba762876
	    [root@56dfba762876 sbin]# ls
	    nginx
	    [root@56dfba762876 sbin]# cd ..
	    [root@56dfba762876 nginx]# ls
	    client_body_temp  conf  fastcgi_temp  html  logs  proxy_temp  sbin  scgi_temp  uwsgi_temp
	    [root@56dfba762876 nginx]# cd c
	    client_body_temp/ conf/             
	    [root@56dfba762876 nginx]# cd conf/
	    [root@56dfba762876 conf]# ls
	    fastcgi.conf          fastcgi_params          koi-utf  mime.types          nginx.conf          scgi_params          uwsgi_params          win-utf
	    fastcgi.conf.default  fastcgi_params.default  koi-win  mime.types.default  nginx.conf.default  scgi_params.default  uwsgi_params.default
	    [root@56dfba762876 conf]# cd ../sbin/             
	    [root@56dfba762876 sbin]# ./nginx -s stop
	    [root@56dfba762876 sbin]# ps -ef|grep nginx
	    root        30     1  0 07:55 pts/0    00:00:00 grep --color=auto nginx

-   修改路由配置

    vi nginx.config


	    [root@56dfba762876 conf]# cd ../sbin/
	    [root@56dfba762876 sbin]# ./nginx
	    [root@56dfba762876 sbin]# cd ..
	    [root@56dfba762876 nginx]# cd conf/
	    [root@56dfba762876 conf]# vi nginx.conf
	    [root@56dfba762876 conf]# cd ../sbin/
	    [root@56dfba762876 sbin]# ./nginx -s stop
	    [root@56dfba762876 sbin]# ./nginx        
	    [root@56dfba762876 sbin]#   

	    server
	    {
	        listen 80;
	        server_name localhost;
	        index index.html index.htm index.php;
	        root /usr/local/nginx/html;

	        location ~ \.php$ {
	            include fastcgi_params;
	            fastcgi_pass unix:/tmp/php-fcgi.sock;
	            fastcgi_index index.php;
	            fastcgi_param SCRIPT_FILENAME /usr/local/nginx/html$fastcgi_script_name;
	        }

	       location /docker-registry-web/ {
	                    proxy_pass   http://192.168.2.102:8000/;
	                    proxy_redirect default;
	            }
	       location /jenkins/ {
	                    proxy_pass   http://192.168.2.102:8002/;
	                    proxy_redirect default;
	            }

	    }

  > jenkins通过nginx映射效果

  ![输入图片说明](https://images.gitee.com/uploads/images/2018/0707/173746_df2818fe_912956.png "屏幕截图.png")

      上图静态资源拉取失败原因：容器内部端口访问不通过本地主机，直接访问资源无法获取，具体原因后续跟进...

<font color='red'>未完待续....</font>  
