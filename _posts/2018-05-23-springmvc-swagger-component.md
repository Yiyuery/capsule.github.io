---
title: Spring MVC 组件配置 之 Swagger整合（自定义样式调整）（springmvc-swagger）
date: 2018-05-23 21:22:00
categories:
- Spring Components
tags:
- spring
- springmvc-swagger
---

笔者的文章中已经有关于springfox和swagger的整合，但是有的项目需要整合静态资源后自定义调整样式，下面提供一种自定义样式的swagger整合方式。
该整合方式对于SpringMVC低版本比较适合、但是swagger的版本已经更新到3.0+了，所以仅供参考和学习。    

---
# Spring MVC 组件配置 之 Swagger整合（自定义样式调整）（springmvc-swagger）

    - swagger静态资源和SpringMVC项目整合
    - 支持自定义样式开发
    - 基于swagger2.2.10版本开发 

## 开发环境
    
    - tomcat 7.0.78
    - jdk 1.7+
    - spring 4.3.13.RELEASE

## SwaggerConfig 配置类

```
package cn.com.showclear.config;

import com.mangofactory.swagger.configuration.SpringSwaggerConfig;
import com.mangofactory.swagger.models.dto.ApiInfo;
import com.mangofactory.swagger.plugin.EnableSwagger;
import com.mangofactory.swagger.plugin.SwaggerSpringMvcPlugin;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.web.servlet.config.annotation.EnableWebMvc;

/**
 * Swagger初始化配置文件
 * @author YF-XIACHAOYANG
 * @date 2017/12/26 10:13
 */
@EnableWebMvc
@EnableSwagger
@ComponentScan(basePackages = "cn.com.showclear.activiti.controller.data")
public class SwaggerConfig {

    private SpringSwaggerConfig springSwaggerConfig;

    /**
     * Required to autowire SpringSwaggerConfig
     */
    @Autowired
    public void setSpringSwaggerConfig(SpringSwaggerConfig springSwaggerConfig)
    {
        this.springSwaggerConfig = springSwaggerConfig;
    }

    /**
     * Every SwaggerSpringMvcPlugin bean is picked up by the swagger-mvc
     * framework - allowing for multiple swagger groups i.e. same code base
     * multiple swagger resource listings.
     */
    @Bean
    public SwaggerSpringMvcPlugin customImplementation()
    {
        return new SwaggerSpringMvcPlugin(this.springSwaggerConfig)
                .apiInfo(apiInfo())
                .includePatterns(".*?");
    }

    private ApiInfo apiInfo()
    {
        ApiInfo apiInfo = new ApiInfo(
                "Scooper Activiti REST-API",
                "工作流后台接口测试",
                "My Apps API terms of service",
                "xiazhaoyang@live.com",
                "web app",
                "My Apps API License URL");
        return apiInfo;
    }
}

```

## spring-mvc.xml

```
<!-- 将 springSwaggerConfig加载到spring容器 -->
<bean class="com.mangofactory.swagger.configuration.SpringSwaggerConfig" />
<!-- 将自定义的swagger配置类加载到spring容器 -->
<bean class="cn.com.showclear.config.SwaggerConfig" />
<!-- don't handle the static resource -->
<mvc:default-servlet-handler></mvc:default-servlet-handler> 
```

## spring-servlet.xml

```
 <!-- 静态资源文件映射，不会被Spring MVC拦截 -->
 <mvc:resources mapping="/swagger/**" location="/swagger/" />
```

## web.xml

```

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
<!--这里可以用 / 但不能用 /*，拦截了所有请求会导致静态资源无法访问，所以要在spring-servlet.xml中配置mvc:resources
    -->
<servlet-mapping>
    <servlet-name>springmvc</servlet-name>
    <url-pattern>/</url-pattern>
</servlet-mapping>
```

## index.html
```
/*添加样式*/
<style type="text/css">
.hide{
  display:none;
}
</style>
/*调整路径*/
 url = "/scooper-activiti/api-docs";
 /*隐藏头部*/
 <div id='header' class="hide">
```

## maven 依赖
```
<!--swagger-->
<dependency>
    <groupId>com.mangofactory</groupId>
    <artifactId>swagger-springmvc</artifactId>
    <version>0.9.5</version>
</dependency>

<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-annotations</artifactId>
    <version>2.4.4</version>
</dependency>

<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.4.4</version>
</dependency>

<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-core</artifactId>
    <version>2.8.9</version>
</dependency>
```

> 项目目录结构

![输入图片说明](https://gitee.com/uploads/images/2018/0523/223715_5ac4ad6b_912956.jpeg "20180111144326047.jpg")

## REFRENCE

    1、swagger整合
    https://www.cnblogs.com/jtlgb/p/6734177.html
    https://www.2cto.com/kf/201604/499072.html
    http://blog.csdn.net/hayre/article/details/51027201
    http://www.mamicode.com/info-detail-525592.html
    
    2、Can't read swagger JSON from http...
    http://blog.csdn.net/shecanwin/article/details/55667102
    http://blog.csdn.net/xyw591238/article/details/51939111
    
    3、No qualifying bean of type 'com.mangofactory.swagger.configuration.SpringSwaggerConfig' available: ...
    https://www.cnblogs.com/driftsky/p/4952918.html
    
    4、swagger文件上传的写法
    http://blog.csdn.net/qq_23167527/article/details/78559096
    
## swagger静态资源文件下载

    1、版本下载列表
    https://github.com/Yiyuery/swagger-ui
    
    2、2.2.10下载链接
    https://github.com/swagger-api/swagger-ui/tree/v2.2.10

> 注意：swagger 版本选择 2.0+ 版本 [2.2.10]


>  效果

![输入图片说明](https://gitee.com/uploads/images/2018/0523/223727_308d8e79_912956.jpeg "20180111144318471.jpg")




