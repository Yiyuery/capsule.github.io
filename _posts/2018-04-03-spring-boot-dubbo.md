---
title: Spring Boot 整合Dubbo开发笔记
date: 2018-04-03 23:56:00
categories:
- Dubbo
tags:
- dubbo
- spring-boot
---

# Spring Boot 整合Dubbo开发笔记

## Dubbo简介

Dubbo是Alibaba开源的分布式服务框架，它最大的特点是按照分层的方式来架构，使用这种方式可以使各个层之间解耦合（或者最大限度地松耦合）。从服务模型的角度来看，Dubbo采用的是一种非常简单的模型，要么是提供方提供服务，要么是消费方消费服务，所以基于这一点可以抽象出服务提供方（Provider）和服务消费方（Consumer）两个角色。关于注册中心、协议支持、服务监控等内容，详见后面描述。

> 具体参见：

    http://shiyanjun.cn/archives/325.html

## Dubbo框架搭建

### 运行环境

    > Spring Boot
    > zookeeper-3.3.6 9
    > dubbo 2.5.4
    > jdk 1.7

### maven依赖

> 项目结构

![这里写图片描述](http://img.blog.csdn.net/20180225164804164?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvc2luYXRfMjg2OTA0MTc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)


> parent-pom

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>cn.com.capsule</groupId>
    <artifactId>spring-boot-dubbo</artifactId>
    <version>1.0.0.0</version>
    <packaging>pom</packaging>


    <properties>
        <!--基本参数集中配置-->
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <java.version>1.7</java.version>
        <!--版本号集中定义-->
        <spring.boot.dubbo.version>1.0.0.0</spring.boot.dubbo.version>
    </properties>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.5.10.RELEASE</version>
        <relativePath/>
    </parent>

    <!--公共依赖-->
    <dependencyManagement>
        <dependencies>
            <!--提供给子类继承的公共依赖-->
        </dependencies>
    </dependencyManagement>


    <!--聚合:合并打包模块集合-->
    <modules>
    </modules>

</project>
```

> api-pom

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>

	<groupId>cn.com.capsule</groupId>
	<artifactId>spring-boot-api</artifactId>
	<version>1.0.0.0</version>
	<packaging>jar</packaging>

	<name>spring-boot-api</name>
	<description>API Demo project for Spring Boot</description>

	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>1.5.10.RELEASE</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>

	<properties>
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
		<project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
		<java.version>1.7</java.version>
	</properties>

	<dependencies>
	</dependencies>

	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>


</project>

```

> provider-pom

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>cn.com.capsule</groupId>
    <artifactId>spring-boot-provider</artifactId>
    <version>1.0.0.0</version>
    <packaging>jar</packaging>

    <name>scpring-boot-provider</name>
    <description>Provider Demo project for Spring Boot</description>

    <parent>
        <groupId>cn.com.capsule</groupId>
        <artifactId>spring-boot-dubbo</artifactId>
        <version>1.0.0.0</version>
        <relativePath/>
    </parent>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.7</java.version>
    </properties>

    <dependencies>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>

        <dependency>
            <groupId>org.springframework.restdocs</groupId>
            <artifactId>spring-restdocs-mockmvc</artifactId>
            <scope>test</scope>
        </dependency>

        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>dubbo</artifactId>
            <version>2.5.6</version>
            <exclusions>
                <exclusion>
                    <artifactId>spring</artifactId>
                    <groupId>org.springframework</groupId>
                </exclusion>
            </exclusions>
        </dependency>

        <dependency>
            <groupId>cn.com.capsule</groupId>
            <artifactId>spring-boot-api</artifactId>
            <version>1.0.0.0</version>
        </dependency>

        <dependency>
            <groupId>org.apache.zookeeper</groupId>
            <artifactId>zookeeper</artifactId>
            <version>3.4.6</version>
            <exclusions>
                <exclusion>
                    <artifactId>slf4j-log4j12</artifactId>
                    <groupId>org.slf4j</groupId>
                </exclusion>
            </exclusions>
        </dependency>

        <dependency>
            <groupId>com.github.sgroschupf</groupId>
            <artifactId>zkclient</artifactId>
            <version>0.1</version>
            <exclusions>
                <exclusion>
                    <artifactId>log4j</artifactId>
                    <groupId>log4j</groupId>
                </exclusion>
                <exclusion>
                    <artifactId>zookeeper</artifactId>
                    <groupId>org.apache.zookeeper</groupId>
                </exclusion>
            </exclusions>
        </dependency>

        <dependency>
            <groupId>org.javassist</groupId>
            <artifactId>javassist</artifactId>
            <version>3.20.0-GA</version>
            <exclusions>
                <exclusion>
                    <artifactId>log4j</artifactId>
                    <groupId>log4j</groupId>
                </exclusion>
                <exclusion>
                    <artifactId>org.apache.zookeeper</artifactId>
                    <groupId>zookeeper</groupId>
                </exclusion>
            </exclusions>
        </dependency>

    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>


</project>

```

> consumer-pom

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>

	<groupId>cn.com.capsule.consumer</groupId>
	<artifactId>spring-boot-comsumer</artifactId>
	<version>1.0.0.0</version>
	<packaging>jar</packaging>

	<name>spring-boot-comsumer</name>
	<description>Demo project for Spring Boot</description>

	<parent>
		<groupId>cn.com.capsule</groupId>
		<artifactId>spring-boot-dubbo</artifactId>
		<version>1.0.0.0</version>
		<relativePath/>
	</parent>

	<properties>
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
		<project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
		<java.version>1.7</java.version>
	</properties>

	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-thymeleaf</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>

		<dependency>
			<groupId>com.alibaba</groupId>
			<artifactId>dubbo</artifactId>
			<version>2.5.6</version>
			<exclusions>
				<exclusion>
					<artifactId>spring</artifactId>
					<groupId>org.springframework</groupId>
				</exclusion>
			</exclusions>
		</dependency>

		<dependency>
			<groupId>cn.com.capsule</groupId>
			<artifactId>spring-boot-api</artifactId>
			<version>1.0.0.0</version>
		</dependency>

		<dependency>
			<groupId>org.apache.zookeeper</groupId>
			<artifactId>zookeeper</artifactId>
			<version>3.4.6</version>
			<exclusions>
				<exclusion>
					<artifactId>slf4j-log4j12</artifactId>
					<groupId>org.slf4j</groupId>
				</exclusion>
			</exclusions>
		</dependency>
		<dependency>
			<groupId>com.github.sgroschupf</groupId>
			<artifactId>zkclient</artifactId>
			<version>0.1</version>
		</dependency>
		<dependency>
			<groupId>org.javassist</groupId>
			<artifactId>javassist</artifactId>
			<version>3.20.0-GA</version>
		</dependency>

	</dependencies>

	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>


</project>

```

## 项目核心代码

    > 接口定义层 spring-boot-api
    
```
package cn.com.capsule.api.services;

/**
 * 接口测试
 * @author YF-XIACHAOYANG
 * @date 2018/2/9 14:16
 */
public interface HiService {

    String hi();
}
```
    > 服务提供层 spring-boot-provider

```
package cn.com.capsule.impl;

import cn.com.capsule.api.services.HiService;
import com.alibaba.dubbo.config.annotation.Service;

/**
 * Dubbo 服务实现
 *
 * @author YF-XIACHAOYANG
 * @date 2018/2/9 14:50
 */
@Service
public class HiServiceImpl implements HiService {

    @Override
    public String hi() {
        return "Hi,Master,I love You! Always!";
    }
```

    > 服务消费（调用）层 sprong-boot-consumer
    
```
package cn.com.capsule.consumer.controller;

import cn.com.capsule.api.services.HiService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.RequestMapping;

import javax.servlet.http.HttpServletRequest;

/**
 * @author YF-XIACHAOYANG
 * @date 2018/2/9 14:24
 */
@org.springframework.stereotype.Controller
@RequestMapping(value="/")
public class HiController {

    @Autowired
    private HiService hiService;

    @RequestMapping(value="/")
    public String hi(HttpServletRequest request, Model model) {

        model.addAttribute("word",hiService.hi());
        return "hi";
    }
}

```
## zookeeper下载和安装
    
    http://mirror.bit.edu.cn/apache/zookeeper/
    
> 基于3.3.6版本

    1、下载解压
    2、找到conf目录下的zoo_sample.cfg
    3、bin同级目录下新增log文件夹（提供日志生成路径）
    4、修改名称为zoo.cfg并调整配置
    5、添加配置
    
```
# The number of milliseconds of each tick
tickTime=2000
# The number of ticks that the initial 
# synchronization phase can take
initLimit=10
# The number of ticks that can pass between 
# sending a request and getting an acknowledgement
syncLimit=5
# the directory where the snapshot is stored.
# dataDir=/tmp/zookeeper
# the port at which the clients will connect

# 这个时间是作为 Zookeeper 服务器之间或客户端与服务器之间维持心跳的时间间隔，也就是每个 tickTime 时间就会发送一个心跳。 
#
syncLimit=5
# 数据存放目录
dataDir=F:\\capsule\\tools\\zookeeper-3.3.6\\data
# 日志存放目录
dataLogDir=F:\\capsule\\tools\\zookeeper-3.3.6\\log
# 注册中心端口号
clientPort=2181
    
```
    6、cmd中使用命令执行zkServer.cmd
    
    

## 项目部署和启动

> 配置调整

spring-boot可以直接通过appication的启动类来启动项目，但是由于内置虚拟Tomcat的默认端口重复绑定冲突，所以在provider或comsumer的application.properties中任一修改端口为server.port=8081即可

> 启动测试

    1、优先开启zookeper
    2、启动provider
    3、启动consumer

> web端调用
    
    根据所配置的consumer的server.port来确定访问路径：
    
    localhost:8080/
    
![image](http://img.blog.csdn.net/20180225164506439?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvc2luYXRfMjg2OTA0MTc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)


## provider和consumer xml

> provider

```
    <?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://code.alibabatech.com/schema/dubbo
       http://code.alibabatech.com/schema/dubbo/dubbo.xsd">

    <!-- 配置可参考 http://dubbo.io/User+Guide-zh.htm -->
    <!-- 服务提供方应用名，用于计算依赖关系 -->
    <dubbo:application name="dubbo-provider" owner="dubbo-provider" />

    <!-- 定义 zookeeper 注册中心地址及协议 -->
    <dubbo:registry protocol="zookeeper" address="127.0.0.1:2181" client="zkclient" />

    <!-- 定义 Dubbo 协议名称及使用的端口，dubbo 协议缺省端口为 20880，如果配置为 -1 或者没有配置 port，则会分配一个没有被占用的端口 -->
    <dubbo:protocol name="dubbo" port="-1" />

    <!-- 声明需要暴露的服务接口 -->
    <dubbo:service interface="cn.com.capsule.api.services.HiService"
                   ref="hiService" timeout="10000" />

    <!-- 和本地 bean 一样实现服务 -->
    <bean id="hiService" class="cn.com.capsule.impl.HiServiceImpl" />

</beans>
```
> consumer

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://code.alibabatech.com/schema/dubbo
       http://code.alibabatech.com/schema/dubbo/dubbo.xsd">

    <!-- 配置可参考 http://dubbo.io/User+Guide-zh.htm -->
    <!-- 消费方应用名，用于计算依赖关系，不是匹配条件，不要与提供方一样 -->
    <dubbo:application name="dubbo-consumer" owner="dubbo-consumer" />

    <!-- 定义 zookeeper 注册中心地址及协议 -->
    <dubbo:registry protocol="zookeeper" address="127.0.0.1:2181" client="zkclient" />

    <!-- 生成远程服务代理，可以和本地 bean 一样使用 testService -->
    <dubbo:reference id="hiService" interface="cn.com.capsule.api.services.HiService" />

</beans>

```
## dubbo部署中发现的小特点

- 1、已注册和订阅过的服务再关闭注册中心后仍然保持正常使用状态
- 2、消费者不用。在提供者挂掉的时候，消费端访问提供者的服务会报错。提供者重启后，只要提供者注册到zookeeper，并且提供者在重启前后的服务签名（方法名、参数、返回值）没有发生变化，客户端仍然可以正常访问提供者的服务
- 3、并不是每次消费者调用服务都会到注册中心去获取服务清单并解析到对应的服务提供地址，它会在内存中缓存一份。如果内存中有就不去注册中获取
- 4、Dubbo目前支持4种注册中心,（multicast zookeeper redis simple）

http://blog.csdn.net/u011659172/article/details/51491518

[点击](https://gitee.com/xiacy/spring-boot-samples/tree/master/spring-boot-dubbo)：获取项目源码

