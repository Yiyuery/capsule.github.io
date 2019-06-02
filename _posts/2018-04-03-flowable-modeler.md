---
title: Flowable 与 modeler 流程设计器整合
date: 2018-04-03 23:22:00
categories:
- Activiti
tags:
- flowable
---

本教程基于Flowable 6.2.1 ，破解 flowable-idm的权限登录，整合SpringMVC实现maven动态导入jar包，期间遇坑无数，写下此文，还望对各位学习工作流的小伙伴有所帮助！

---

# Flowable 与 modeler 流程设计器整合

本教程基于Flowable 6.2.1 ，破解 flowable-idm的权限登录，整合SpringMVC实现maven动态导入jar包，期间遇坑无数，写下此文，还望对各位学习工作流的小伙伴有所帮助！

---

## 准备工作

> 1、flowable-modeler 获取

![这里写图片描述](http://img.blog.csdn.net/20180312104650575?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvc2luYXRfMjg2OTA0MTc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
[下载链接>>>](https://flowable.org/downloads.html)

> 2、资源文件解压并导入SpringMVC项目

![这里写图片描述](http://img.blog.csdn.net/20180312105000595?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvc2luYXRfMjg2OTA0MTc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

```
/** 配置静态资源路由*/
<!-- don't handle the static resource -->
<mvc:default-servlet-handler />
<!-- if you use annotation you must configure following setting -->
<mvc:annotation-driven />

<!-- 静态资源文件映射，不会被Spring MVC拦截 -->
<mvc:resources mapping="/js/**" location="/static/js/"/>
<mvc:resources mapping="/css/**" location="/static/css/"/>
<mvc:resources mapping="/plugin/**" location="/static/plugin/"/>
<mvc:resources mapping="/images/**" location="/static/images/"/>
<mvc:resources mapping="/resource/**" location="/static/resource/"/>
<mvc:resources mapping="/html/**" location="/static/html/"/>
<mvc:resources mapping="/swagger/**" location="/swagger/"/>
<mvc:resources mapping="/modeler/**" location="/modeler/"/>
```
> 3、调整flowable-modeler前端访问路径

![这里写图片描述](http://img.blog.csdn.net/20180312105906755?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvc2luYXRfMjg2OTA0MTc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

> 4、模拟登录接口（破解idm的权限校验）

- 后台登录（可在此处向己方人员系统做切换，前端缓存人员token信息或是利用session皆可）

```
package cn.com.showclear.activiti.controller.data;

import cn.com.showclear.common.resp.RespMapJson;
import com.wordnik.swagger.annotations.Api;
import com.wordnik.swagger.annotations.ApiOperation;
import org.flowable.idm.pojo.AppUser;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;


/**
 * 模拟登录
 * @author YF-XIACHAOYANG
 * @date 2018/1/30 11:18
 */
@RestController
@RequestMapping("/data/modeler/")
public class FlowableModelerRestController {

    @RequestMapping("account")  
    public RespMapJson account() {
        AppUser user = new AppUser("2017","activiti","scooper","scooper-activiti");
        return new RespMapJson().setData(user);
    }
}

```
- 前端登录接口替换

![这里写图片描述](http://img.blog.csdn.net/20180312110352471?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvc2luYXRfMjg2OTA0MTc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

## app-rest接口导入和破解

由于flowable-modeler的流程设计器页面很多操作会访问后台接口，在非maven的框架下，有人是通过导入jar包来实现的，在maven的框架下，我采用导入jar包源码并覆盖扫描的方式来实现后台servlet接口的实现。

> 1、pom

```
<!--流程设计器-->
<dependency>
    <groupId>org.flowable</groupId>
    <artifactId>flowable-ui-modeler-rest</artifactId>
    <version>${flowable.version}</version>
</dependency>

<!--路由配置-->
<dependency>
    <groupId>org.flowable</groupId>
    <artifactId>flowable-ui-modeler-conf</artifactId>
    <version>${flowable.version}</version>
</dependency>
```
> 2、源码覆盖二度扫描注入

解决扫描注入失败的问题，解决部分接口和配置文件加载路径调整的问题，其他可能涉及到需要修改接口的问题。

	> 源码导入（方便复写）

![这里写图片描述](http://img.blog.csdn.net/20180312111028250?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvc2luYXRfMjg2OTA0MTc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)


	> web.xml 添加路由映射

```
/*在web.xml中添加路由映射*/
<!-- 配置Spring核心控制器 -->
<servlet>
     <servlet-name>springmvc</servlet-name>
     <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
     <!--可以不配置.默认所对应的配置文件是WEB-INF下的{servlet-name}-servlet.xml，这里便是：spring-servlet.xml
     -->
     <init-param>
         <param-name>contextConfigLocation</param-name>
         <param-value>classpath:/config/spring/spring-servlet.xml</param-value>
     </init-param>
     <load-on-startup>1</load-on-startup>
 </servlet>
 <!--这里可以用 / 但不能用 /*，拦截了所有请求会导致静态资源无法访问，所以要在spring-servlet.xml中配置mvc:resources-->
 <servlet-mapping>
     <servlet-name>springmvc</servlet-name>
     <url-pattern>/</url-pattern>
 </servlet-mapping>

 <!-- 配置floeable-app-rest控制器 -->
 <servlet>
     <servlet-name>appDispatcherServlet</servlet-name>
     <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
     <!--可以不配置.默认所对应的配置文件是WEB-INF下的{servlet-name}-servlet.xml，这里便是：spring-servlet.xml
     -->
     <init-param>
         <param-name>contextConfigLocation</param-name>
         <param-value>classpath:/config/flowable/app-servlet.xml</param-value>
     </init-param>
     <load-on-startup>1</load-on-startup>
 </servlet>
 <servlet-mapping>
     <servlet-name>appDispatcherServlet</servlet-name>
     <url-pattern>/app/*</url-pattern>
 </servlet-mapping>
```

	> 添加配置：扫描注入

![这里写图片描述](http://img.blog.csdn.net/20180312111527309?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvc2luYXRfMjg2OTA0MTc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

	> 汉化：主要是修改stencilset_bpmn.json文件（百度一下可以找到很多）

![这里写图片描述](http://img.blog.csdn.net/20180312111737357?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvc2luYXRfMjg2OTA0MTc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

## 效果

![这里写图片描述](http://img.blog.csdn.net/20180312111851915?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvc2luYXRfMjg2OTA0MTc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

![这里写图片描述](http://img.blog.csdn.net/20180312111859637?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvc2luYXRfMjg2OTA0MTc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

![这里写图片描述](http://img.blog.csdn.net/20180312111907496?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvc2luYXRfMjg2OTA0MTc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

## 备注

	1、数据库操作请使用c3p0，因为flowable-modeler中使用的是这个，避免冲突。
	2、由于静态资源在项目中，所以样式的修改完全可以自定义。

> 参考

```

 基本知识

    1、c3p0 Spring 配置
    http://blog.csdn.net/l1028386804/article/details/51162560
    2、Flowable基础五 Flowable 数据库配置
    http://www.shareniu.com/article/98.htm
    3、[ github ] flowable-engine
    https://github.com/flowable/flowable-engine/tree/flowable-6.2.1
    4、flowable使用
    http://www.shareniu.com/article/19.htm
    5、flowable 集成spring boot
    http://www.shareniu.com/article/81.htm
    6、Flowable基础十四 Flowable modeler汉化
    http://www.shareniu.com/article/108.htm
    7、Flowable无法登录
    http://www.shareniu.com/article/174.htm
    8、flowable 官网
    http://www.flowable.org/
    9、activiti自定义流程之Spring整合activiti-modeler实例（一）：环境搭建
    http://blog.csdn.net/hj7jay/article/details/51149026
    10、如何整合Flowable-modeler到自己的项目中
    http://veevv.com/2017/03/17/flowable-modeler-integrate/
    11、springboot整合flowable
    http://blog.csdn.net/zl1zl2zl3/article/details/78921162
    12、flowable与modeler整合
    http://www.shareniu.com/article/52.htm
    13、取消验证
    http://veevv.com/2017/03/17/flowable-modeler-integrate/
    14、国际化
    https://www.cnblogs.com/liukemng/p/3750117.html


 通用

    1、maven 源码
    http://www.mvnjar.com/

    2、手册
    https://tkjohn.github.io/flowable-userguide/

```

## 更多

> 扫码关注“架构探险之道”，回复`文章名称`获取更多源码和文章资源

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190403222309957.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzI4NjkwNDE3,size_16,color_FFFFFF,t_70)

> 知识星球(扫码加入获取源码和文章资源链接)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190403222322267.jpeg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzI4NjkwNDE3,size_16,color_FFFFFF,t_70)
