---
title: Docker 之 Spring Boot 自动化部署
date: 2018-07-04 16:22:03
categories:
- Docker
tags:
- docker
- jenkins
---

&lt;架构探险之路> Docker搭建微服务自动部署平台，让我们来看看如何利用spring boot 的maven插件实现代码的自动化构建和发布吧！

* * *

# Docker 之 Spring Boot 自动化部署

## spring-boot-maven-plugin

> 插件配置

    <properties>
      ...
      <!--Docker Repo-->
      <docker.registry>127.0.0.1:5000</docker.registry>
    </properties>

    ...

    <build>
           <plugins>
              <plugin>
                   <groupId>com.spotify</groupId>
                   <artifactId>docker-maven-plugin</artifactId>
                   <version>1.1.1</version>
                   <configuration>
                       <imageName>${docker.registry}/${project.groupId}/${project.artifactId}:${project.version}</imageName>
                       <dockerDirectory>
                           ${project.build.outputDirectory}
                       </dockerDirectory>
                       <resources>
                           <resource>
                               <directory>${project.build.directory}</directory>
                               <include>${project.build.finalName}.jar</include>
                           </resource>
                       </resources>
                   </configuration>
               </plugin>
          </plugins>
    </build>

## Dockfile

> Dockerfile文件放在resources下

    FROM 127.0.0.1:5000/env-jdk:1.0
    MAINTAINER "xiachaoyang"<xiazhaoyang@live.com>
    ADD msa-api-hello-0.0.1.jar app.jar
    EXPOSE 8080
    CMD java -jar app.jar

> maven

  编译 > 打包 > 创建镜像

    mvn clean package docker:build

  推送镜像

    mvn docker:push

![输入图片说明](https://images.gitee.com/uploads/images/2018/0704/223024_451a6e61_912956.png "屏幕截图.png")

  查看仓库`127.0.0.1:8000`

![输入图片说明](https://images.gitee.com/uploads/images/2018/0704/223140_ccdde0f3_912956.png "屏幕截图.png")

    docker images

![输入图片说明](https://images.gitee.com/uploads/images/2018/0704/223301_14c2c687_912956.png "屏幕截图.png")

## 整合gitlab、jenkins实现自动化发布

[Docker 之 GitLab 局域网代码托管](http://xiazhaoyang.tech/docker/2018/07/07/docker-gitlab/)
[Docker 之 Jenkins自动化部署](http://xiazhaoyang.tech/docker/2018/07/05/docker-jenkins/)
