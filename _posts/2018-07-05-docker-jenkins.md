---
title: Docker 之 Jenkins自动化部署
date: 2018-07-05 16:22:04
categories:
- Docker
tags:
- docker
- jenkins
---

&lt;架构探险之路> Docker搭建微服务自动部署平台，让我们来看看如何实现基于Docker的Jenkins自动化部署。

* * *

# Docker 之 Jenkins自动化部署

> 构建思路

-   Docker 安装jenkins,用来拉取代码自动更新
-   Docker 安装gitlab，用来局域网或本地管理代码
-   Docker 安装本地镜像仓库registry、docker-register-web
-   Spring Boot 开发代码后编写Dokcerfile文件
-   Spring Boot 利用docker的mvn插件测试镜像的生成和推送
-   测试镜像运行

    * * *

    镜像的自动构建分两种情况：

    > jenkins所在容器中已部署docker服务

      直接在构建中利用shell脚本完成Dokcerfile文件的复制和执行，进而在jenkins所在容器内完成镜像的构建

    > jenkins所在容器中未部署docker服务

    -   jenkins中利用Docker插件实现镜像构建
    -   jenkins 全局工具配置中安装docker[自动安装]
    -   将jenkins部署在宿主机上，重复上述关联步骤。gitlab可切换为github、gitee

> 为了提升镜像的自动构建速度，最终采用本地部署jenkins的方式，因为宿主机是有docker运行环境的。

## Jenkins部署

> jenkins环境 [相对版本较低]

    docker pull jenkins

    docker run -d -p 8002:8080 -v ~/jenkins:/var/jenkins_home --name jenkins --restart=always jenkins

    查看容器日志

    docker logs -f jenkins

    查看容器运行

    docker ps

![输入图片说明](https://images.gitee.com/uploads/images/2018/0705/084140_862102d2_912956.png "屏幕截图.png")

    界面访问`127.0.0.1:8080`,自动跳转至登录界面

![输入图片说明](https://images.gitee.com/uploads/images/2018/0705/084615_6234d78d_912956.png "屏幕截图.png")

    观察日志获取初始密码

![输入图片说明](https://images.gitee.com/uploads/images/2018/0705/084855_ae63f145_912956.png "屏幕截图.png")

    jenkins安装

![输入图片说明](https://images.gitee.com/uploads/images/2018/0705/084956_5b85274a_912956.png "屏幕截图.png")

    访问
    admin:jenkins

> jenkinsci/jenkins [最新版本]

-   镜像

    docker pull jenkinsci/jenkins

-   启动

    docker run -d -p 8002:8080 -m 1024m -v ~/jenkins:/var/jenkins_home --name jenkins --restart=always jenkinsci/jenkins

-   日志

    docker logs -f jenkins

-   jenkis 绑定gitlab

    docker run -d -p 8002:8080 -m 1024m -v ~/jenkins:/var/jenkins_home --name jenkins --restart=always --link gitlab:gitlab.yiyuery.com jenkinsci/jenkins

-   maven构建项目

    > gitlab's project 配置

    ![输入图片说明](https://images.gitee.com/uploads/images/2018/0708/145910_510d945e_912956.png "屏幕截图.png")

      此处ssh鉴权失败需要生成key添加到gitlab中，和github一样,例外，不能使用ssh，只能用http
      点击Add,输入账户`root`,密码Abc23++，此为gitlab管理员账号和访问gitlab时设置的密码

    > 构建后的maven命令配置

    ![输入图片说明](https://images.gitee.com/uploads/images/2018/0708/150503_784d65a9_912956.png "屏幕截图.png")

      下方的为构建后的需要存档的文件配置 [Ant风格]
      `pom.xml`文件对应工程目录`msa-api-hello/pom.xml`

    > 构建结果

    ![输入图片说明](https://images.gitee.com/uploads/images/2018/0708/145419_2ebc03fa_912956.png "屏幕截图.png")

    > 本地映射jenkins工作空间

![输入图片说明](https://images.gitee.com/uploads/images/2018/0708/152714_c5aaa55a_912956.png "屏幕截图.png")

    > 定时构建

![输入图片说明](https://images.gitee.com/uploads/images/2018/0708/153254_fdb58023_912956.png "屏幕截图.png")
    表示每10分钟执行一次，用H不用\*，是为了降低同一时间执行多个构建所带来的性能开销，使用H可以将具体的构建时间进行Hash

-   shell脚本自动化构建Docker镜像

    > 可用环境变量

    ![输入图片说明](https://images.gitee.com/uploads/images/2018/0708/161355_63c54625_912956.png "屏幕截图.png")

```
    # 定义变量
    API_NAME="msa-api-hello"
    API_VERSION="0.0.1"
    API_PORT="8101"
    IMAGE_NAME="127.0.0.1:5000/com.msa/$API_NAME:$BUILD_NUMBER"
    CONTAINER_NAME=$API_NAME-$API_VERSION

    # 进入target目录并复制Dockerfile文件
    cd $WORKSPACE/target
    cp classes/Dockerfile .

    # 构建Docker镜像
    docker build -t $IMAGE_NAME .

    # 推送Docker镜像
    docker push $IMAGE_NAME

    # 删除Docker容器
    cid=$(docker ps | grep $CONTAINER_NAME |awk '{print $1}')
    if [ x"$cid" != x ]
    	then
       	docker rm -f $cid
    fi

    # 启动Docker容器
    docker run -d -p $API_PORT:8080 --name $CONTAINER_NAME $IMAGE_NAME

    # 删除Dockerfile文件
    rm -f Dockerfile
```

![输入图片说明](https://images.gitee.com/uploads/images/2018/0708/155135_4d07d8dc_912956.png "屏幕截图.png")

> 提升maven构建速度

  maven clean install -Dmaven.test.skip=true
  跨过测试类的执行

> jenkins 无法通过shell脚本进行docker镜像的构建

![输入图片说明](https://images.gitee.com/uploads/images/2018/0708/164418_6303bc80_912956.png "屏幕截图.png")

  解决方案：

    - 不使用任何Jenkins镜像，宿主机安装Jenkins [宿主机有Docker服务]
    - 不使用官方Jenkins镜像，自己构造带有Docker服务的Jenkins镜像
    - Docker-in-Docker [DinD]
    - Docker-outside-of-Docker [DooD]
    - 使用Jenkins的Docker插件

* * *

## 自动构建并发布

  考虑到本地笔记本开发环境，多个dokcer的运行效率本来就低，因此，为提高构建速度，下载war包后在本地tomcat中运行，需要对jenkins进行构建的话，启动tomcat即可。

> tomcat 部署项目

  直接放在tomcat的webapp目录下后在bin目录下直接启动也是可以的。此处主要是因为idea中开发演示项目，直接放在一起，方便管理。

![输入图片说明](https://images.gitee.com/uploads/images/2018/0715/104116_f53df3ef_912956.png "屏幕截图.png")

> 安装maven插件

  不安装插件则无法构建maven项目，jenkins默认是不支持maven的

![输入图片说明](https://images.gitee.com/uploads/images/2018/0715/103852_108efbc2_912956.png "屏幕截图.png")
![输入图片说明](https://images.gitee.com/uploads/images/2018/0715/104030_32d92011_912956.png "屏幕截图.png")

> 配置后拉取项目代码进行构建

![输入图片说明](https://images.gitee.com/uploads/images/2018/0715/103547_345e476e_912956.png "屏幕截图.png")

![输入图片说明](https://images.gitee.com/uploads/images/2018/0715/103629_bd4e19d8_912956.png "屏幕截图.png")

tag使用的是构建次数作为版本标记

> 自动发布

![输入图片说明](https://images.gitee.com/uploads/images/2018/0715/143538_c8498d9b_912956.png "屏幕截图.png")

-   仓库

    ![输入图片说明](https://images.gitee.com/uploads/images/2018/0715/143731_7b31750e_912956.png "屏幕截图.png")

-   运行

    ![输入图片说明](https://images.gitee.com/uploads/images/2018/0715/143844_c6d296e2_912956.png "屏幕截图.png")

`备注：`

-   初次构建速度比较慢，后面由于镜像缓存、maven依赖的下载完成，构件速度会变快很多。
-   shell脚本遇到问题请自行学习相关知识
-   轻量级微服务的自动化发布平台，主要实现思路：Jenkins从GitLab中获取源码，构建后生成docker镜像，以Docker容器的方式进行发布，此外，我还将生成的Docker镜像推送到本地的Docker Registry，以供生产环境使用。如此，我们交付的不再是源码，而是Docker镜像，这种方式更加简单高效。

## REFERENCES

1.  [Jenkins Wiki](https://wiki.jenkins.io/display/JENKINS/Installing+Jenkins+with+Docker)
2.  [Jenkins 安装教程](http://www.cnblogs.com/stulzq/p/8627360.html)
3.  [Jenkins 利用maven、git管理项目](https://jingyan.baidu.com/album/597a06433ff422312a52436f.html?picindex=1)
4.  [Jenkins与Docker相关的Plugin使用](https://blog.csdn.net/ztsinghua/article/details/52128140)
5.  [宿主机安装jenkins方案](https://www.jianshu.com/p/a7d7df97fe4b)
6.  [Shell菜鸟教程](http://www.runoob.com/linux/linux-shell-test.html)


## 微信公众号

<center>
<img src="https://images.gitee.com/uploads/images/2018/0717/215030_8e782063_912956.png" width="50%" height="50%"/>
</center>

扫码关注或搜索`架构探险之道`获取最新文章，不积跬步无以至千里，坚持每周一更，坚持技术分享。我和你们一起成长 ^_^ ！
