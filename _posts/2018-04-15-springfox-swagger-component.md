---
title: Spring MVC 组件配置 之 RESTFUL API文档以及Mock应用（springfox-swagger）
date: 2018-04-15 21:22:00
categories:
- Spring Components
tags:
- spring
- springfox-swagger
---

Swagger是世界上最大的框架API开发工具的API规范（OAS），从设计、文档、测试和部署上努力使整个API开发生命周期大大缩短。    

---

# Spring MVC 组件配置 之 RESTFUL API文档以及Mock应用（springfox-swagger）

## 动态API管理

    - Swagger是世界上最大的框架API开发工具的API规范（OAS），从设计、文档、测试和部署上努力使整个API开发生命周期大大缩短。
    - 随着互联网技术的发展，现在的网站架构基本都由原来的后端渲染，变成了：前端渲染、前后端分离的形态。
    - 微服务REST-API的方式链接前端和后台。
具体的可以看这里[API 的撰写 - 契约](https://mp.weixin.qq.com/s?__biz=MzA3NDM0ODQwMw==&mid=402114651&idx=1&sn=a7b891f532e29b73afd83f17ae071023&scene=1&srcid=0331zejNfNvZ5ccJEdBpJxIr&from=singlemessage&isappinstalled=0#wechat_redirect)

![输入图片说明](https://gitee.com/uploads/images/2018/0415/204725_ad0549ad_912956.png "20170827202033991.png")
    
## 整合方式

> maven 

```
        <!--springfox-swagger start-->
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger2</artifactId>
            <version>${springfox.version}</version>
        </dependency>

        <!-- 无需引入额外静态资源-->
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger-ui</artifactId>
            <version>${springfox.version}</version>
        </dependency>

        <!--jackson用于将springfox返回的文档对象转换成JSON字符串-->
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-annotations</artifactId>
            <version>${version.jackson}</version>
        </dependency>

        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-databind</artifactId>
            <version>${version.jackson}</version>
        </dependency>

        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-core</artifactId>
            <version>${version.jackson}</version>
        </dependency>

        <!--petStore是官方提供的一个代码参考, 可用于后期写文档时进行参考, 可不加-->
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-petstore</artifactId>
            <version>${springfox.version}</version>
        </dependency>
        <!--springfox-swagger end-->
```

> 静态资源配置（2种方式）

    - spring-mvc.xml 中添加如下配置
    
```
    <!-- don't handle the static resource -->
    <mvc:default-servlet-handler/>
```
    
    - spring-swagger.xml 中添加如下配置
    
```
    <!--jar中静态资源配置访问权限-->
    <mvc:resources mapping="swagger-ui.html" location="classpath:/META-INF/resources/"/>
    <mvc:resources mapping="/webjars/**" location="classpath:/META-INF/resources/webjars/"/>
```

> web.xml中配置ContextListener

```xml

    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>
            classpath:config/spring/spring-*.xml
        </param-value>
    </context-param>
    
    <servlet>
        <servlet-name>springmvc</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <!--可以不配置.默认所对应的配置文件是WEB-INF下的{servlet-name}-servlet.xml，这里便是：springmvc-servlet.xml-->
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:config/spring/spring-servlet.xml</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>
    <!--这里用 / 不拦截静态资源，若是用/*，拦截了所有请求会导致静态资源无法访问，需要在spring-servlet.xml中配置mvc:resources-->
    <servlet-mapping>
        <servlet-name>springmvc</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>
    
    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>
        
        
    
```

> Controller控制器和SwaggerConfiguration配置类的扫描（或bean标签实例）

```
    <!--spring-beans.xml-->
    <!--Controller层-->
    <context:component-scan base-package="com.capsule.swagger.controller"/>
    <!--Configuration-->
    <!--swagger support[实例配置项]-->
    <context:component-scan base-package="com.capsule.common.config"/>

```

> Controller 测试类 和 SwaggerConfiguration配置类

    - Controller

```java
package com.capsule.swagger.controller.data;

import com.capsule.common.resp.RespMapJson;
import com.capsule.swagger.pojo.UserDo;
import io.swagger.annotations.Api;
import io.swagger.annotations.ApiOperation;
import io.swagger.annotations.ApiParam;
import io.swagger.annotations.ApiResponse;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

/**
 * <p></p>
 *
 * @author Yiyuery
 * @version V1.0
 * @Description:
 * @Date 2018/4/15 10:14
 */
@RestController
@RequestMapping("/user/")
@Api(value = "User控制器")
public class UserController {

    @ApiOperation(value = "根据用户id查询用户信息", httpMethod = "GET") //produces = "application/json" 设置返回类型
    @ApiResponse(code = 200, message = "success", response = RespMapJson.class)
    @RequestMapping(path = "hi", method = RequestMethod.GET)
    public RespMapJson test(@ApiParam(name = "userId", required = true, value = "用户Id") @RequestParam("userId") int userId) {
        return new RespMapJson().setData(new UserDo().setName("Yiyuery").setAge(25).setId(userId)).setMsg("用户信息获取成功！");
    }

    @RequestMapping(path = "getUserName", method = RequestMethod.GET)
    public String getUserName() {
        return "Yiyuery";
    }

}

```    
    
    - SwaggerConfiguration
    
```java
package com.capsule.common.config;

import com.google.common.base.Predicate;
import com.google.common.base.Predicates;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import springfox.documentation.builders.ApiInfoBuilder;
import springfox.documentation.builders.PathSelectors;
import springfox.documentation.service.ApiInfo;
import springfox.documentation.service.Contact;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spring.web.plugins.Docket;
import springfox.documentation.swagger2.annotations.EnableSwagger2;

import static com.google.common.base.Predicates.or;
import static springfox.documentation.builders.PathSelectors.regex;

@Configuration
@EnableSwagger2
public class SwaggerConfiguration {

    @Bean
    public Docket apiDocket() {
        return new Docket(DocumentationType.SWAGGER_2)
                .groupName("API")
                .apiInfo(apiInfo())
                .select()
                .paths(apiPathsRex())
                .build().pathMapping("/rest1");
    }

    @Bean
    public Docket userDocket() {
        return new Docket(DocumentationType.SWAGGER_2)
                .groupName("USER")
                .apiInfo(apiInfo())
                .select()                
                //.apis(RequestHandlerSelectors.basePackage("com.capsule.web2.controller")) 指定controller扫描位置
                .paths(userPathsRex())
                .build().pathMapping("/rest2");
    }

    @Bean
    public Docket defaultDocket() {
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .select()
                .paths(defaultPathsRex())
                .build().pathMapping("/");
    }

    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                .title("SWAGGER REST FOR PART OF API")
                .version("1.0-SNAPSHOT")
                .description("外部接口文档")
               .termsOfServiceUrl("http://xiazhaoyang.tech")
               .contact(new Contact("xiazhaoyang","","xiazhaoyang@live.com"))
                .license("Apache 2.0")
                .licenseUrl("http://www.apache.org/licenses/LICENSE-2.0.html")
                .build();
    }

    private Predicate<String> apiPathsRex() {
        return or(
                regex("/api.*")
        );
    }

    private Predicate<String> userPathsRex() {
        return or(
                regex("/user.*")
        );
    }

    private Predicate<String> defaultPathsRex() {
        return or(
                regex("/user.*"),
                regex("/api.*")
        );
    }
}


``` 

> 注解说明

    @ApiModel 表明这是一个被swagger框架管理的model，用于实体类上

    @ApiModelProperty model对象中成员变量名称标注，常用属性value,name

    @ApiModel的class的属性上，这里的value是对字段的描述，example是取值例子，注意这里的example很有用，对于前后端开发工程师理解文档起到了关键的作用，因为会在api文档页面上显示出这些取值来；这个注解还有一些字段取值，可以自己研究，举例说一个：position，表明字段在model中的顺序

    @ApiOperation标注在具体请求方法上，value和notes的作用差不多，都是对请求进行说明；tags则是对请求进行分类的，比如你有好几个controller，分别属于不同的功能模块，那这里我们就可以使用tags来区分了，看上去很有条理
    
    @Api()用于类名标注，常用属性value,description,tag 
    
    @ApiImplicitParams 接口、请求方法上方对入参的描述，通常用({@ApiImplicitParam(...),...})包裹多个参数
    
    @ApiImplicitParam 接口、请求方法上方基本参数描述
    
    @ApiParam(value="...",name="...",type="int") 方法中调用的参数描述，紧邻参数，默认body application/json，可以通过加@RequestParam("...")Integer ... 来指定参数类型
     
 
    

[Swagger注解](https://huawei-servicecomb.gitbooks.io/developerguide/content/build-provider/swagger-annotation.html) 

```java

/*入参*/
@ApiModel(description = "用户请求表单")
public class UserForm {
    @ApiModelProperty(value = "姓名", example = "maxTse",position = 1)
    private String username;
    //省略getter、setter
}

/*返回参数*/
@JsonInclude(JsonInclude.Include.NON_NULL)
@ApiModel
public class RespResult<T> implements Serializable {
    @ApiModelProperty(value = "数据", example = "")
    public T data;
    @ApiModelProperty(value = "状态码,0表示成功 其他表示失败", example = "0")
    public int status;
    @ApiModelProperty(value = "错误信息", example = "操作成功")
    public String message = "";
    //...
}

/*控制器*/
@Controller
@RequestMapping(value = "/test", consumes = "application/json", produces = "application/json")
public class TestController {
private static final Logger LOGGER = LoggerFactory.getLogger(TestController.class);
    @ApiOperation(value = "swagger test", notes = "swagger test first", tags = {SwaggerConstants.TEST_TAG})
    @ResponseBody
    @RequestMapping(value = "/first", method = RequestMethod.POST)
    public RespResult<String> first(@RequestBody UserForm userForm) {
        LOGGER.info("first userForm={}", userForm);
        return RespResult.successResult(userForm.getUsername());
    }
}

```

官方 springfox-petstore 中也有大量的范例，idea中载入源码即可了解所有标签的使用


    - swagger的注解主要是为了界面和json中对完成对接口的描述
    - swagger的注解在controller层加会污染代码，如何优化？
        > Controller继承接口API,在API中利用 swagger标签进行描述
        
```java

@Api(value = "User控制器")
public interface UserActionAPI {

    @ApiOperation(value = "根据用户id查询用户信息", httpMethod = "GET",produces = "application/json",consumes = "application/json")
    @ApiResponse(code = 200, message = "success", response = RespMapJson.class)
    @ApiImplicitParams({
            @ApiImplicitParam(value="用户Id",defaultValue="1",name="userId",required=true,paramType="query",dataType="int")})
    RespMapJson test(@ApiParam(name = "userId", required = true, value = "用户Id") @RequestParam("userId") int userId);
}

@RestController
@RequestMapping("/user/")
public class UserController implements UserActionAPI {


    @RequestMapping(path = "hi", method = RequestMethod.GET)
    public RespMapJson test(int userId) {
        return new RespMapJson().setData(new UserDo().setName("Yiyuery").setAge(25).setId(userId)).setMsg("用户信息获取成功！");
    }

    @RequestMapping(path = "getUserName", method = RequestMethod.GET)
    public String getUserName() {
        return "Yiyuery";
    }

}

```   

> 接口json
    
    - 默认路径

     localhost[/ip]:port+${ctx}+web.xml[中配置的DispatchServlet拦截路径]（'/'）+/v2/api-docs+[group='your api group def']
    
    - 自定义API接口组  http://localhost:8080/springfox-swagger/v2/api-docs?group=API 
    
![输入图片说明](https://gitee.com/uploads/images/2018/0415/213820_95a8068a_912956.png "201804152138.png")
    
    - 默认接口组  http://localhost:8080/springfox-swagger/v2/api-docs

![输入图片说明](https://gitee.com/uploads/images/2018/0415/213852_84e5bcf3_912956.png "201804152139.png") 
     

> rest-web效果

![输入图片说明](https://gitee.com/uploads/images/2018/0415/214757_d146cb9f_912956.png "201804152148.png")


## 文档生成

    1、web.xml需要添加swagger的servlet然后做url mapping，不然swagger-ui.html和/v2/api-docs显示不出来。
    2、swagger的配置文件要加一个swagger config的bean。
    3、swagger config类是做定制化显示用的。例如版本号，作者信息等等。通过实现多个Docket可以输出多条api路径信息和接口文档。
    4、使用springfox-staticdocs可实现输出静态api文档。但是有个坑，默认编码并非utf-8，中文会乱码。只好自己实现一个Swagger2MarkupResultHandler，在里面指定编码格式。已提pr，不知他们接不接受。


## 前端对接





## REFRENCE

> springfox

[springfox使用手册](https://springfox.github.io/springfox/docs/snapshot/#getting-started)

[springfox-swagger整合教程](http://www.baeldung.com/swagger-2-documentation-for-spring-rest-api)

[SpringMVC集成springfox-swagger2自动生成接口文档](https://www.cnblogs.com/zhaojiankai/p/8318359.html)



```
    https://blog.csdn.net/qq_16256793/article/details/79522749
```
