---
title: Spring MVC 项目xml配置 之 视图解析器（ViewResolver）
date: 2018-04-15 16:22:00
categories:
- Spring Configuration
tags:
- spring
---

当我们对SpringMVC控制的资源发起请求时，这些请求都会被SpringMVC的DispatcherServlet处理，接着Spring会分析看哪一个HandlerMapping定义的所有请求映射中存在对该请求的最合理的映射。然后通过该HandlerMapping取得其对应的Handler，接着再通过相应的HandlerAdapter处理该Handler。HandlerAdapter在对Handler进行处理之后会返回一个ModelAndView对象。在获得了ModelAndView对象之后，Spring就需要把该View渲染给用户，即返回给浏览器。在这个渲染的过程中，发挥作用的就是ViewResolver和View。当Handler返回的ModelAndView中不包含真正的视图，只返回一个逻辑视图名称的时候，ViewResolver就会把该逻辑视图名称解析为真正的视图View对象。View是真正进行视图渲染，把结果返回给浏览器的。    

---


# Spring MVC 项目xml配置 之 视图解析器（ViewResolver）

## 视图解析器

> 前言

当我们对SpringMVC控制的资源发起请求时，这些请求都会被SpringMVC的DispatcherServlet处理，接着Spring会分析看哪一个HandlerMapping定义的所有请求映射中存在对该请求的最合理的映射。然后通过该HandlerMapping取得其对应的Handler，接着再通过相应的HandlerAdapter处理该Handler。HandlerAdapter在对Handler进行处理之后会返回一个ModelAndView对象。在获得了ModelAndView对象之后，Spring就需要把该View渲染给用户，即返回给浏览器。在这个渲染的过程中，发挥作用的就是ViewResolver和View。当Handler返回的ModelAndView中不包含真正的视图，只返回一个逻辑视图名称的时候，ViewResolver就会把该逻辑视图名称解析为真正的视图View对象。View是真正进行视图渲染，把结果返回给浏览器的。    



### xml配置

> 解析当前项目webapp下的/views中所有后缀为.html的资源文件

```
    <!-- 默认定义视图解析器 -->
    <!-- 默认定义视图解析器 -->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="order" value="1"/>
        <property name="prefix" value="/views/"/>
        <property name="suffix" value=".html"/>
    </bean>
```
     - 核心方法

```
        //org.springframework.web.servlet.view.InternalResourceViewResolver
	@Override
	protected AbstractUrlBasedView buildView(String viewName) throws Exception {
		InternalResourceView view = (InternalResourceView) super.buildView(viewName);
		if (this.alwaysInclude != null) {
			view.setAlwaysInclude(this.alwaysInclude);
		}
		view.setPreventDispatchLoop(true);
		return view;
	}
```

> 以整合Freemarker为例，多视图解析器自动解析为例

     - maven

```
 <!--freemarker support [依赖于spring-context-support]实际项目中上文已经引入，此处取消关联依赖-->
        <dependency>
            <groupId>org.freemarker</groupId>
            <artifactId>freemarker</artifactId>
            <version>2.3.23</version>
            <exclusions>
                <exclusion>
                    <artifactId>org.springframework</artifactId>
                    <groupId>spring-context-support</groupId>
                </exclusion>
            </exclusions>
        </dependency>
```

    - spring-freemarker.xml

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <description>Freemarker 配置</description>

    <!--配置从文件读取-->
    <bean id="freemarkConfig" class="org.springframework.beans.factory.config.PropertiesFactoryBean">
        <property name="location" value="classpath:config/components/freemarker.properties"/>
    </bean>

    <!-- 配置FreeMark视图解析器 -->
    <bean id="freeMarkerViewResolver" class="org.springframework.web.servlet.view.freemarker.FreeMarkerViewResolver">
        <property name="contentType" value="text/html;charset=UTF-8"/>
        <property name="viewClass" value="org.springframework.web.servlet.view.freemarker.FreeMarkerView"/>
        <property name="suffix" value=".html"/>
        <property name="cache" value="false"/>
        <property name="exposeSessionAttributes" value="true"/>
        <property name="exposeRequestAttributes" value="true"/>
        <property name="exposeSpringMacroHelpers" value="true"/>
        <!-- 在页面中使用${rc.contextPath}就可获得contextPath -->
        <!--<property name="requestContextAttribute" value="rc"/>-->
        <property name="order" value="0"/>
    </bean>

    <bean id="fmXmlEscape" class="freemarker.template.utility.XmlEscape"/>

    <bean id="freeMarkerConfigurer" class="org.springframework.web.servlet.view.freemarker.FreeMarkerConfigurer">
        <property name="templateLoaderPath" value="/static/html/"/>
        <property name="defaultEncoding" value="UTF-8"/>
        <property name="freemarkerSettings" ref="freemarkConfig"/>
        <!--手动配置
        <property name="freemarkerSettings">
             <props>
                 <prop key="template_update_delay">3600</prop>
                 <prop key="locale">zh_CN</prop>
                 <prop key="datetime_format">yyyy-MM-dd HH:mm:ss</prop>
                 <prop key="date_format">yyyy-MM-dd</prop>
                 <prop key="number_format">#.##</prop>
             </props>
         </property>
        -->
        <property name="freemarkerVariables">
            <map>
                <entry key="xml_escape" value-ref="fmXmlEscape"/>
            </map>
        </property>
    </bean>

</beans>
```
    > templateLoaderPath为Freemarker需要进行视图解析的文件路径，可以选择手动配置或是从配置文件读取

### 视图跳转控制和自动解析

> ViewController

```
@Controller
@RequestMapping("/view/")
public class ViewController {

    @RequestMapping(path = "/hi", method = RequestMethod.GET)
    public String test() {
        return "hi";
    }

    @RequestMapping(path = "/vi", method = RequestMethod.GET)
    public String vi() {
        return "view-test";
    }
}

```
> 视图路由解析

    - webapp 目录结构

![输入图片说明](https://gitee.com/uploads/images/2018/0415/152035_99e820ee_912956.png "201804151521.png")

    - 自动选择视图解析器后进行跳转

![输入图片说明](https://gitee.com/uploads/images/2018/0415/152248_f87d218f_912956.png "201804151523.png")

![输入图片说明](https://gitee.com/uploads/images/2018/0415/152330_ffc78566_912956.png "201804151524.png")


### 补充说明

    - 视图解析器调用顺序

order越小，优先级越高.

    - freemarker 模版路径支持配置多个

 ```
<property name="templateLoaderPaths">  
    <list>  
        <value>/views/</value>  
        <value>/ftl/</value>  
    </list>  
</property>  
```

    - 视图解析器类型分析
     > AbstractCachingViewResolver
     > UrlBasedViewResolver
     > InternalResourceViewResolver
     > XmlViewResolver
     > BeanNameViewResolver
     > ResourceBundleViewResolver
     > FreeMarkerViewResolver、VolocityViewResolver    

SpringMVC用于处理视图最重要的两个接口是ViewResolver和View。ViewResolver的主要作用是把一个逻辑上的视图名称解析为一个真正的视图，SpringMVC中用于把View对象呈现给客户端的是View对象本身，而ViewResolver只是把逻辑视图名称解析为对象的View对象。View接口的主要作用是用于处理视图，然后返回给客户端。   

      - 视图解析器链

 在SpringMVC中可以同时定义多个ViewResolver视图解析器，然后它们会组成一个ViewResolver链。当Controller处理器方法返回一个逻辑视图名称后，ViewResolver链将根据其中ViewResolver的优先级来进行处理。所有的ViewResolver都实现了Ordered接口，在Spring中实现了这个接口的类都是可以排序的。 **在ViewResolver中是通过order属性来指定顺序的，默认都是最大值** 。所以我们可以通过指定ViewResolver的order属性来实现ViewResolver的优先级，order属性是Integer类型，order越小，对应的ViewResolver将有越高的解析视图的权利，所以第一个进行解析的将是ViewResolver链中order值最小的那个。当一个ViewResolver在进行视图解析后返回的View对象是null的话就表示该ViewResolver不能解析该视图，这个时候如果还存在其他order值比它大的ViewResolver就会调用剩余的ViewResolver中的order值最小的那个来解析该视图，依此类推。当ViewResolver在进行视图解析后返回的是一个非空的View对象的时候，就表示该ViewResolver能够解析该视图，那么视图解析这一步就完成了，后续的ViewResolver将不会再用来解析该视图。当定义的所有ViewResolver都不能解析该视图的时候，Spring就会抛出一个异常。
基于Spring支持的这种ViewResolver链模式，我们就可以在SpringMVC应用中同时定义多个ViewResolver，给定不同的order值，这样我们就可以对特定的视图特定处理，以此来支持同一应用中有多种视图类型。注意：像 **InternalResourceViewResolver** 这种能解析所有的视图，即永远能返回一个非空View对象的ViewResolver一定要把它放在 **ViewResolver链的最后面** 。      


### REFERENCE

[SpringMVC之视图解析器及解析过程浅析](https://blog.csdn.net/zmx729618/article/details/51554762)


## 更多

> 扫码关注“架构探险之道”，回复`文章名称`获取更多源码和文章资源

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190403222309957.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzI4NjkwNDE3,size_16,color_FFFFFF,t_70)

> 知识星球(扫码加入获取源码和文章资源链接)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190403222322267.jpeg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzI4NjkwNDE3,size_16,color_FFFFFF,t_70)
