---
title: Docker 之 CentOS 环境安装
date: 2018-06-21 16:22:01
categories:
- Docker
tags:
- docker
---

&lt;架构探险之路> Docker搭建微服务自动部署平台，让我们先来了解下CentOS 环境安装、以及Docker的一些常用操作。

* * *

# Docker 之 CentOS 环境安装

> 版本信息

-   docker -v

          xiazhaoyangdeMacBook-Pro:~ xiazhaoyang$ docker -v
          Docker version 18.05.0-ce, build f150324

-   uname -a

          [root@f7106025b994 /]# uname -a
          Linux f7106025b994 4.9.93-linuxkit-aufs #1 SMP Wed Jun 6 16:55:56 UTC 2018 x86_64 x86_64 x86_64 GNU/Linux

-   本地磁盘挂载(资源文件共享)

        docker run -i -t -v ~/software:/mnt/software centos /bin/bash

        xiazhaoyangdeMacBook-Pro:/ xiazhaoyang$ cd ~/software
        xiazhaoyangdeMacBook-Pro:software xiazhaoyang$ ls
        jdk-8u172-linux-x64.tar.gz
        xiazhaoyangdeMacBook-Pro:software xiazhaoyang$ pwd
        /Users/xiazhaoyang/software

-   centos软件安装目录 /opt

## 安装jdk

> 解压

     tar -zxvf /mnt/software/jdk-8u172-linux-x64.tar.gz  -C /opt

> 创建映射

     ln -s /opt/jdk1.8.0_172 /opt/jdk

    # 测试映射路径
    ll /opt/jdk
    /opt/jdk/bin/java -version

> 修改环境变量

     vi /etc/profile

    # JAVA_ENV_CONFIGURATION
    export JAVA_HOME=/opt/jdk              
    export JRE_HOME=${JAVA_HOME}/jre
    export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib  
    export PATH=${JAVA_HOME}/bin:$PATH

> 激活配置并测试

  source /etc/profile

    [root@7941a20a85b6 opt]# java -version
    java version "1.8.0_172"
    Java(TM) SE Runtime Environment (build 1.8.0_172-b11)
    Java HotSpot(TM) 64-Bit Server VM (build 25.172-b11, mixed mode)

## 镜像操作

> 镜像上传

    # 将本地容器提交为镜像
    xiazhaoyangdeMacBook-Pro:software xiazhaoyang$ docker commit 7941a20a85b6 xiachaoyang/centos-java
    sha256:905af87b4a5efcd0d7c5128700393d65f50c4575878cb250fae177c987d87964
    xiazhaoyangdeMacBook-Pro:software xiazhaoyang$ docker images
    REPOSITORY                TAG                 IMAGE ID            CREATED             SIZE
    xiachaoyang/centos-java   latest              905af87b4a5e        18 seconds ago      587MB
    centos                    latest              49f7960eb7e4        2 weeks ago         200MB
    xiazhaoyangdeMacBook-Pro:software xiazhaoyang$

> 镜像验证

        xiazhaoyangdeMacBook-Pro:~ xiazhaoyang$ docker images
        REPOSITORY                TAG                 IMAGE ID            CREATED             SIZE
        xiachaoyang/centos-java   latest              905af87b4a5e        10 minutes ago      587MB
        centos                    latest              49f7960eb7e4        2 weeks ago         200MB
        xiazhaoyangdeMacBook-Pro:~ xiazhaoyang$ docker run --rm xiachaoyang/centos-java /opt/jdk/bin/java -version
        java version "1.8.0_172"
        Java(TM) SE Runtime Environment (build 1.8.0_172-b11)
        Java HotSpot(TM) 64-Bit Server VM (build 25.172-b11, mixed mode)
        xiazhaoyangdeMacBook-Pro:~ xiazhaoyang$

<font color=red>[备注]</font>

    1.  容器通过commit直接生成的镜像使用`docker run -i -t xiachaoyang/centos-java`启动镜像后,需要再次执行source /etc/profile 才能激活环境变量

        [root@288652cb51f3 /]# java -version
        bash: java: command not found
        [root@288652cb51f3 /]# source /etc/profile
        [root@288652cb51f3 /]# java -version
        java version "1.8.0_172"
        Java(TM) SE Runtime Environment (build 1.8.0_172-b11)
        Java HotSpot(TM) 64-Bit Server VM (build 25.172-b11, mixed mode)
        [root@288652cb51f3 /]#

推荐使用Dockerfile创建镜像

    2.  启动镜像后可以多终端访问同一运行中容器

        # 启动一个实例
        xiazhaoyangdeMacBook-Pro:~ xiazhaoyang$ docker run -i -t xiachaoyang/centos-java /bin/bash
        [root@288652cb51f3 /]#

        # 开启新终端查询实例ID并进入
        xiazhaoyangdeMacBook-Pro:~ xiazhaoyang$ docker ps
        CONTAINER ID        IMAGE                     COMMAND             CREATED             STATUS              PORTS               NAMES
        288652cb51f3        xiachaoyang/centos-java   "/bin/bash"         2 minutes ago       Up About a minute                       nifty_bhabha
        xiazhaoyangdeMacBook-Pro:~ xiazhaoyang$ docker attach 288652cb51f3
        [root@288652cb51f3 /]#

        # 测试运行中实例是否有变化
        xiazhaoyangdeMacBook-Pro:~ xiazhaoyang$ docker ps
        CONTAINER ID        IMAGE                     COMMAND             CREATED             STATUS              PORTS               NAMES
        288652cb51f3        xiachaoyang/centos-java   "/bin/bash"         2 minutes ago       Up 2 minutes                            nifty_bhabha
        xiazhaoyangdeMacBook-Pro:~ xiazhaoyang$

    3.  多个终端链接同一镜像，退出后整体退出。
    所有操作都是并行的，A输入cmd1,B也同样出现cmd1

![输入图片说明](https://gitee.com/uploads/images/2018/0622/081846_9db202dc_912956.png "屏幕截图.png")

    4.  基于centos进行的镜像操作，本地centos是否会发生变化，能否提交覆盖

    不能,推送类似于git操作，需要权限。

        xiazhaoyangdeMacBook-Pro:~ xiazhaoyang$ docker push centos
        The push refers to repository [docker.io/library/centos]
        bcc97fbfc9e1: Layer already exists
        errors:
        denied: requested access to the resource is denied
        unauthorized: authentication required

    5.  对于本地库中镜像的操作(运行容器并修改部分东西)，能否覆盖提交，能否提交到仓库同名下，还是说需要利用tag进行标记？

可以覆盖更新镜像，参考下文:

[创建本地镜像(commit、Dockerfile)](https://blog.csdn.net/u010246789/article/details/54139168)

    6.  同一镜像生成的不同ID的容器，互相之间不受影响，修改在退出后也不会保存

> 镜像推送

[Docker Hub](https://hub.docker.com/)

-   创建DockerHub账号
-   创建仓库
-   推送本地镜像

    > 非必须，没有权限时需要登录

    docker login

    > 推送

    docker push xiachaoyang/env-jdk


        xiazhaoyangdeMacBook-Pro:~ xiazhaoyang$ docker images
        REPOSITORY            TAG                 IMAGE ID            CREATED             SIZE
        xiachaoyang/env-jdk   latest              abe21e6640f3        13 hours ago        587MB
        centos                latest              49f7960eb7e4        2 weeks ago         200MB
        xiazhaoyangdeMacBook-Pro:~ xiazhaoyang$ docker push xiachaoyang/env-jdk
        The push refers to repository [docker.io/xiachaoyang/env-jdk]
        f65468761332: Pushed
        5ed6bb2dd837: Pushed
        bcc97fbfc9e1: Mounted from library/centos
        latest: digest: sha256:a7f0324eae38d9ce41389bccd43c92f3fa9d8a1a1dbe5ff52d6be978815fb267 size: 949

-   搜索镜像

    docker search xiaochaoyang/env-jdk

    Docker Hub会定时对已上传的镜像仓库进行索引，随后可通过`docker search xiachaoyang/centos-java`进行搜索自己推送到Docker Hub的镜像仓库

> 镜像功能测试

        # 查询运行中容器
        xiazhaoyangdeMacBook-Pro:~ xiazhaoyang$ docker ps
        CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
        xiazhaoyangdeMacBook-Pro:~ xiazhaoyang$ docker run -i -t xiachaoyang/centos-java /bin/bash
        [root@288652cb51f3 /]#

> Dockerfile 创建镜像

        xiazhaoyangdeMacBook-Pro:Dockerfile         jdk-8u172-linux-x64.tar.gz
        xiazhaoyangdeMacBook-Pro:jdk-8u172-linux-x64 xiazhaoyang$ docker build -t xiachaoyang/centos-java .
        Sending build context to Docker daemon  190.9MB
        Step 1/7 : FROM centos:latest
        ---> 49f7960eb7e4
        Step 2/7 : MAINTAINER  "xiachaoyang"<xiazhaoyang@live.com>
        ---> Using cache
        ---> c1009b593364
        Step 3/7 : ADD jdk-8u172-linux-x64.tar.gz /opt
        ---> Using cache
        ---> 9c32f5eacb71
        Step 4/7 : RUN ln -s /opt/jdk-8u172-linux-x64 /opt/jdk
        ---> Running in 23a59c89c7be
        Removing intermediate container 23a59c89c7be
        ---> 99b3d5d9446c
        Step 5/7 : ENV JAVA_HOME /opt/jdk
        ---> Running in f70d6dd4dda2
        Removing intermediate container f70d6dd4dda2
        ---> 685cd05aa266
        Step 6/7 : ENV PATH $JAVA_HOME/bin:$PATH
        ---> Running in 934a8279744d
        Removing intermediate container 934a8279744d
        ---> 75f551548c7a
        Step 7/7 : CMD java -version
        ---> Running in 9ff68222bc64
        Removing intermediate container 9ff68222bc64
        ---> f1992f5f7b4c
        Successfully built f1992f5f7b4c
        Successfully tagged xiachaoyang/centos-java:latest

> 标记版本信息

        xiazhaoyangdeMacBook-Pro:~ xiazhaoyang$ docker images
        REPOSITORY                TAG                 IMAGE ID            CREATED             SIZE
        xiachaoyang/centos-java   latest              f1992f5f7b4c        2 minutes ago       587MB
        xiachaoyang/centos-java   <none>              905af87b4a5e        10 hours ago        587MB
        centos                    latest              49f7960eb7e4        2 weeks ago         200MB
        xiazhaoyangdeMacBook-Pro:~ xiazhaoyang$ docker tag 905af87b4a5e xiachaoyang/centos-java:1.0
        xiazhaoyangdeMacBook-Pro:~ xiazhaoyang$ docker images
        REPOSITORY                TAG                 IMAGE ID            CREATED             SIZE
        xiachaoyang/centos-java   latest              f1992f5f7b4c        3 minutes ago       587MB
        xiachaoyang/centos-java   1.0                 905af87b4a5e        10 hours ago        587MB
        centos                    latest              49f7960eb7e4        2 weeks ago         200MB
        xiazhaoyangdeMacBook-Pro:~ xiazhaoyang$

> 提交(运行中容器提交为镜像)

  查看运行中容器

    docker ps

  将当前容器提交为一个新的镜像

    docker commit [容器ID] xaichaoyang/centos-java

  查看镜像

    docker images

> 镜像删除

-   容器删除

    dokcer rm [容器ID]

-   容器批量删除

    `-f`表示强制删除

         docker rm -f $(docker ps -a -q)
         docker rm -f `docker ps -a -q`


-   镜像删除

        docker rmi [镜像ID]

-   批量删除

            docker rmi -f  `docker images -a -q`

    > Dockerfile 踩坑

-   解压文件名

      jdk文件解压文件名非原tgz的文件名，导致ln命令的符号链接显示红色闪烁，是由于文件不存在导致，调整Dockerfile构建文件后可以正常生成


        # 运行镜像测试
        xiazhaoyangdeMacBook-Pro:jdk-8u172-linux-x64 xiazhaoyang$ docker run --rm xiachaoyang/env-jdk java -version
        java version "1.8.0_172"
        Java(TM) SE Runtime Environment (build 1.8.0_172-b11)
        Java HotSpot(TM) 64-Bit Server VM (build 25.172-b11, mixed mode)

> 查看容器列表信息

  docker container ls -a

-   批量删除

    docker rm -f `docker ps -a -q`

## 搭建Docker Registry（本地镜像仓库）

-   本地搭建

    > 下载并运行registry

    docker run -d -p 5000:5000 -v ~/docker-registry:/tmp/registry --name local-docker-res  --restart=always registry

    \-d : 表示后台启动该容器
    \-p : 表示对容器中应用程序暴露的端口号进行端口映射，冒号左边的端口(50000)为宿主机的端口，冒号右边的端口(5000)为容器内部需要报录的端口
    \-v : 数据卷选项，表示将宿主机的~/docker-registry 目录映射为容器的/tmp/registry
    \-restart：docker服务重启后总是重启此容器


        xiazhaoyangdeMacBook-Pro:~ xiazhaoyang$ docker run -d -p 50000:5000 -v ~/docker-registry:/tmp/registry --name local_res registry
        5c08afe92712746db4fb48e02b5b274e205e7dce3cb38395846ca57b5d387c5e
        xiazhaoyangdeMacBook-Pro:~ xiazhaoyang$ docker ps
        CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                     NAMES
        5c08afe92712        registry            "/entrypoint.sh /etc…"   27 seconds ago      Up 26 seconds       0.0.0.0:50000->5000/tcp   determined_banach

  访问`http://127.0.0.1:5000/v2/_catalog?n=100`查看仓库

![输入图片说明](https://gitee.com/uploads/images/2018/0628/074208_29beb849_912956.png "屏幕截图.png")

    > 下载web服务
    docker run -it -p 8000:8080 --restart=always --name local-docker-res-web --link local-docker-res  -e REGISTRY_URL=http://local-docker-res:5000/v2 -e REGISTRY_NAME=localhost:5000 hyper/docker-registry-web

    `8000:8080` 内置web服务是8080端口的，将内部8080端口映射到本机的8000端口上。
    `REGISTRY_URL` 修改本地仓库域名
    > 运行时遇到的问题
    1. 8000：8000,内部8080启动失败：

    - 查看运行中容器
    docker ps
    - 关闭容器
    docker stop [容器ID]
    - 容器搜索
    docker container ls -a
    - 删除容器
    docker rm [容器ID]
    - 再次运行web服务

    2. 仓库名称写错
    `I/O error on GET request for "http://registry-srv:5000/v2/_catalog?n=100":registry-srv; nested exception is java.net.UnknownHostException: registry-srv`
    参考网上资料修改时未换成本地自定义仓库名local-docker-res
    修改命令重新执行


    - 操作时对于容器的状态还原

    批量关闭容器
    docker stop `docker ps -a -q`

    批量删除容器
    docker rm -f `docker ps -a -q`

    删除镜像  
    docker rmi -f [镜像ID]

> 本地仓库web服务

-   界面效果

    访问`http://127.0.0.1:8000/`查看仓库web界面

![输入图片说明](https://gitee.com/uploads/images/2018/0627/084528_1210837e_912956.png "屏幕截图.png")

-   测试本地仓库推送

> 可能出现的push失败

    $ docker push 127.0.0.1:5000/xiachaoyang/env-jdk
    The push refers to a repository [127.0.0.1:5000/xiachoayang/env-jdk]
    Get https://127.0.0.1:5000/v1/_ping: http: server gave HTTP response to HTTPS client

这是因为Docker在1.3.x之后默认docker registry使用的是https，为了解决这个问题，Linux中修改本地主机的docker启动配置文件，添加

    --insecure-registry 127.0.0.1:5000

Docker for Mac中在图形化终端上修改如下，填写后“Apply&Restart”。

 ![输入图片说明](https://gitee.com/uploads/images/2018/0628/075348_73a1395f_912956.png "屏幕截图.png")

    docker images
    REPOSITORY                            TAG                 IMAGE ID            CREATED             SIZE
    127.0.0.1:50000/xiachaoyang/env-jdk   latest              abe21e6640f3        2 days ago          587MB
    xiachaoyang/env-jdk                   latest              abe21e6640f3        2 days ago          587MB
    centos                                latest              49f7960eb7e4        3 weeks ago         200MB
    registry                              latest              d1fd7d86a825        5 months ago        33.3MB
    hyper/docker-registry-web             latest              0db5683824d8        20 months ago       599MB
    xiazhaoyangdeMacBook-Pro:~ xiazhaoyang$ docker tag abe21e6640f3  127.0.0.1:5000/xiachaoyang/env-jdk
    xiazhaoyangdeMacBook-Pro:~ xiazhaoyang$ docker push 127.0.0.1:5000/xiachaoyang/env-jdk
    The push refers to repository [127.0.0.1:5000/xiachaoyang/env-jdk]
    f65468761332: Pushed
    5ed6bb2dd837: Pushing [===============>                                   ]  120.4MB/387.4MB
    bcc97fbfc9e1: Pushing [=======================>                           ]  94.75MB/199.7MB

-   查看推送结果

![输入图片说明](https://gitee.com/uploads/images/2018/0627/085153_d505cca3_912956.png "屏幕截图.png")

-   删除镜像后测试本地仓库


    docker images
    REPOSITORY                  TAG                 IMAGE ID            CREATED             SIZE
    centos                      latest              49f7960eb7e4        3 weeks ago         200MB
    registry                    latest              d1fd7d86a825        5 months ago        33.3MB
    hyper/docker-registry-web   latest              0db5683824d8        20 months ago       599MB
    xiazhaoyangdeMacBook-Pro:~ xiazhaoyang$ docker pull 127.0.0.1:5000/xiachaoyang/env-jdk
    Using default tag: latest
    latest: Pulling from xiachaoyang/env-jdk
    7dc0dca2b151: Already exists
    71ed27c83409: Pull complete
    b7e1a9577e86: Pull complete
    Digest: sha256:a7f0324eae38d9ce41389bccd43c92f3fa9d8a1a1dbe5ff52d6be978815fb267
    Status: Downloaded newer image for 127.0.0.1:5000/xiachaoyang/env-jdk:latest

-   运行容器


    xiazhaoyangdeMacBook-Pro:~ xiazhaoyang$ docker images
    REPOSITORY                           TAG                 IMAGE ID            CREATED             SIZE
    127.0.0.1:5000/xiachaoyang/env-jdk   latest              abe21e6640f3        2 days ago          587MB
    centos                               latest              49f7960eb7e4        3 weeks ago         200MB
    registry                             latest              d1fd7d86a825        5 months ago        33.3MB
    hyper/docker-registry-web            latest              0db5683824d8        20 months ago       599MB
    xiazhaoyangdeMacBook-Pro:~ xiazhaoyang$ docker run --rm 127.0.0.1:5000/xiachaoyang/env-jdk java -version
    java version "1.8.0_172"
    Java(TM) SE Runtime Environment (build 1.8.0_172-b11)
    Java HotSpot(TM) 64-Bit Server VM (build 25.172-b11, mixed mode)
    xiazhaoyangdeMacBook-Pro:~ xiazhaoyang$

<font color="red">docker restart 【容器ID】重新命令会保留挂参启动的配置信息 </font>

    xiazhaoyangdeMacBook-Pro:~ xiazhaoyang$ docker ps
    CONTAINER ID        IMAGE                       COMMAND             CREATED             STATUS              PORTS                    NAMES
    9b17f2a0c10b        hyper/docker-registry-web   "start.sh"          23 hours ago        Up 23 hours         0.0.0.0:8000->8080/tcp   local-docker-res-web
    xiazhaoyangdeMacBook-Pro:~ xiazhaoyang$ docker container ls -a
    CONTAINER ID        IMAGE                       COMMAND                  CREATED             STATUS                     PORTS                    NAMES
    9b17f2a0c10b        hyper/docker-registry-web   "start.sh"               23 hours ago        Up 23 hours                0.0.0.0:8000->8080/tcp   local-docker-res-web
    353ec8eb8c91        registry                    "/entrypoint.sh /etc…"   23 hours ago        Exited (2) 2 minutes ago                            local-docker-res
    xiazhaoyangdeMacBook-Pro:~ xiazhaoyang$ docker restart 353ec8eb8c91
    353ec8eb8c91
    xiazhaoyangdeMacBook-Pro:~ xiazhaoyang$ docker container ls -a
    CONTAINER ID        IMAGE                       COMMAND                  CREATED             STATUS              PORTS                    NAMES
    9b17f2a0c10b        hyper/docker-registry-web   "start.sh"               23 hours ago        Up 23 hours         0.0.0.0:8000->8080/tcp   local-docker-res-web
    353ec8eb8c91        registry                    "/entrypoint.sh /etc…"   23 hours ago        Up 3 seconds        0.0.0.0:5000->5000/tcp   local-docker-res
    xiazhaoyangdeMacBook-Pro:~ xiazhaoyang$

-   查看本地磁盘挂载中镜像

    若未找到本地镜像位置：需了解Docker在Mac中的数据卷挂载相关知识

## 镜像源配置

> Mac下修改镜像源和阿里云镜像加速

-   阿里云注册和购买
-   产品和服务中选择容器镜像服务

[Aliyun提供的Docker for Mac 安装包下载目录](http://mirrors.aliyun.com/docker-toolbox/mac/docker-for-mac/)

![](https://gitee.com/uploads/images/2018/0627/081441_03ab822f_912956.png)

> 如何配置镜像加速器

右键点击桌面顶栏的 docker 图标，选择 Preferences ，在 Daemon 标签（Docker 17.03 之前版本为 Advanced 标签）下的 Registry mirrors 列表中将`https://xxxxx.mirror.aliyuncs.com`加到"registry-mirrors"的数组里，点击 Apply & Restart按钮，等待Docker重启并应用配置的镜像加速器。

    docker info

    # http://hub-mirror.c.163.com/为网易docker镜像源

    Insecure Registries:
     127.0.0.0/8
    Registry Mirrors:
     http://hub-mirror.c.163.com/
     https://xxxx.mirror.aliyuncs.com/
    Live Restore Enabled: false

  配置后可以明显加快镜像拉取速度.

## REFERENCES

1.  [Dockerfile构建镜像](https://blog.csdn.net/qinyushuang/article/details/43342553)
2.  [Docker私有仓库搭建](https://www.jianshu.com/p/9cf9d1c8b00c)
3.  [Dcoker数据卷挂载](https://yeasy.gitbooks.io/docker_practice/content/data_management/volume.html)


## 更多

> 扫码关注“架构探险之道”，回复`文章名称`获取更多源码和文章资源

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190403222309957.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzI4NjkwNDE3,size_16,color_FFFFFF,t_70)

> 知识星球(扫码加入获取源码和文章资源链接)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190403222322267.jpeg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzI4NjkwNDE3,size_16,color_FFFFFF,t_70)
