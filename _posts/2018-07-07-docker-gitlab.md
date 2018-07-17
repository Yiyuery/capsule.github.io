---
title: Docker 之 GitLab 局域网代码托管
date: 2018-07-07 16:22:06
categories:
- Docker
tags:
- docker
- gitlab
---

&lt;架构探险之路> Docker搭建微服务自动部署平台，让我们来了解下基于Docker的gitlab局域网代码托管吧！

* * *

# Docker 之 GitLab 局域网代码托管

## 部署

-   拉取镜像

    docker pull gitlab/gitlab-ce

-   本地域名DNS映射配置

    sudo vi /etc/hosts
    添加 127.0.0.1 gitlab.yiyuery.com

-   运行

    docker run -d -m 1024m -h gitlab.yiyuery.com -p 22:22 -p 80:80 -v ~/gitlab/etc:/etc/gitlab -v ~/gitlab/log:/var/log/gitlab -v ~/gitlab/opt:/var/opt/gitlab --name gitlab --restart=always gitlab/gitlab-ce

`此处需注意关闭其他占用80端口的进程`

    域名的映射默认使用的是80端口

-   git 操作

    创建gitlab工程

![输入图片说明](https://images.gitee.com/uploads/images/2018/0708/100245_fe2789c0_912956.png "屏幕截图.png")

  参考上图完成推送代码到远端[gitlab仓库]

![输入图片说明](https://images.gitee.com/uploads/images/2018/0708/152004_e20c3a06_912956.png "屏幕截图.png")



## 微信公众号

<center>
<img src="https://images.gitee.com/uploads/images/2018/0717/215030_8e782063_912956.png" width="50%" height="50%"/>
</center>

扫码关注或搜索`架构探险之道`获取最新文章，坚持每周一更，坚持技术分享的我和你们一起成长 ^_^ ！
