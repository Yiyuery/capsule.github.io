---
title: Docker 之 运行状态监控
date: 2018-07-06 16:22:00
categories:
- Docker
tags:
- docker
---

Docker搭建微服务自动部署 <架构探险之路>，让我们先来了解下Docker运行中的状态监控和内存控制吧！

---

# Docker 之 运行状态监控

## 调整Docker容器内存

> 查看当前内存占用情况

    docker stats

    CONTAINER ID        NAME                   CPU %               MEM USAGE / LIMIT     MEM %               NET I/O             BLOCK I/O           PIDS
    fad28b1084ef        jenkins                0.25%               432.6MiB / 1.952GiB   21.64%              2.51MB / 3.04MB     53.4MB / 9.42MB     38
    56dfba762876        nginx      0.00%               692KiB / 1.952GiB     0.03%               788B / 0B           205kB / 0B          1
    d75f05b4d7f8        local-docker-res-web   0.36%               525.6MiB / 1.952GiB   26.29%              65.4kB / 318kB      104MB / 4.1kB       34
    1b5239bc943b        local-docker-res       0.03%               6.367MiB / 1.952GiB   0.32%               47.2kB / 37.7kB     13.5MB / 0B         7

  可以通过添加-m来控制启东时的容器内存限制：

-   批量停止容器

        docker stop `docker ps  -q`

-   批量删除容器

        docker rm -f `docker container ls -a -q`

-   启动容器

    > registry


    docker run -d -m 1024m -p 5000:5000 -v ~/docker-registry:/tmp/registry --name local-docker-res  
    --restart=always registry

> nginx

    xiazhaoyangdeMacBook-Pro:~ xiazhaoyang$ docker run -i -m 1024m -t -p 8000:80 --name nginx --restart=always env-nginx
    [root@cfa1a6786510 /]# ps -ef|grep nginx
    root        16     1  0 10:01 pts/0    00:00:00 grep --color=auto nginx
    [root@cfa1a6786510 /]# cd /usr/local/nginx/sbin/
    [root@cfa1a6786510 sbin]# ./nginx
    [root@cfa1a6786510 sbin]# ./nginx -s stop
    [root@cfa1a6786510 sbin]# vi ../conf/nginx.conf
    [root@cfa1a6786510 sbin]# ./nginx
    [root@cfa1a6786510 sbin]#

> docker-registry-web

    docker run -it -m 1024m -p 8001:8080 --restart=always  --name local-docker-res-web  --link local-docker-res  -e  REGISTRY_URL=http://local-docker-res:5000/v2 -e  REGISTRY_NAME=localhost:5000 hyper/docker-registry-web

> jenkins

    docker run -d -m 1024m  -p 8002:8080 -v ~/jenkins:/var/jenkins_home --link github:github.com --name jenkins --restart=always jenkins

-   再次查看运行状态

```
CONTAINER ID        NAME                   CPU %               MEM USAGE / LIMIT   MEM %               NET I/O             BLOCK I/O           PIDS
cb7a3cfdeafb        jenkins                113.15%             283.7MiB / 1GiB     27.70%              648B / 0B           11.1MB / 9.31MB     26
cfa1a6786510        nginx                  0.03%               6.793MiB / 1GiB     0.66%               928B / 0B           385kB / 24.6kB      4
84ad7ef849b3        local-docker-res-web   0.49%               521.4MiB / 1GiB     50.92%              928B / 0B           74.5MB / 4.1kB      24
190e85388e0d        local-docker-res       0.00%               3.625MiB / 1GiB     0.35%               1.11kB / 0B         1.29MB / 0B         7

```
