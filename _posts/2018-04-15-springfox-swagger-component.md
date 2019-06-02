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

>具体的可以看这里
[API 的撰写 - 契约](https://mp.weixin.qq.com/s?__biz=MzA3NDM0ODQwMw==&mid=402114651&idx=1&sn=a7b891f532e29b73afd83f17ae071023&scene=1&srcid=0331zejNfNvZ5ccJEdBpJxIr&from=singlemessage&isappinstalled=0#wechat_redirect)

![输入图片说明](https://gitee.com/uploads/images/2018/0415/204725_ad0549ad_912956.png "20170827202033991.png")

## SpringMVC 整合方式

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
                //pathMapping("/xcy-web")会统一在该接口分组下的接口前加上前缀 "/xcy-web"
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

> 标签描述定义 & 多路径接口扫描

```
@Bean
public Docket apiDocket() {
    return (new Docket(DocumentationType.SWAGGER_2)).groupName("api")        
    .produces(this.produce())
    .pathMapping("/api")
    .globalOperationParameters(this.setHeaderToken())
    .apiInfo(this.apiInfo())
    .tags(new Tag("权限接口", "菜单、区域、组织、资源等权限判断"), new Tag[]{new Tag("用户接口", "按条件查询用户、修改密码、获取加密参数"), new Tag("认证接口", "WEB登录、客户端登录、自动登录、获取验证码"), new Tag("数据迁移接口", "海燕数据迁移至海豚")})
    .select()
    .apis(Predicates.or(new Predicate[]{RequestHandlerSelectors.basePackage("com.xxx.xxx.privilege.api"), RequestHandlerSelectors.basePackage("com.xxx.xxx.user.api"), RequestHandlerSelectors.basePackage("com.xxx.xxx.userauth.api")}))
    .build();
}

@Bean
public Docket uiDocket() {
    return (new Docket(DocumentationType.SWAGGER_2))
    .groupName("ui")
    .produces(this.produce())
    .pathMapping("/ui")
    .apiInfo(this.uiInfo())
    .tags(new Tag("用户接口", "用户添加、查询、修改、删除、导入导出等接口"), new Tag[]{new Tag("用户组接口", "用户组添加、修改、删除、查询等接口"), new Tag("用户实名人员信息获取接口", "获取人员组织相关的接口"), new Tag("AD域接口", "获取Windows域组织架构以及用户接口"), new Tag("角色相关接口", "添加角色、修改角色、删除角色、查询角色等相关接口"), new Tag("角色权限配置接口", "查询角色权限、查询资源类型、更新角色权限等相关接口")})
    .select()
    .apis(Predicates.or(RequestHandlerSelectors.basePackage("com.xxx.xxx.privilege.role.action"), RequestHandlerSelectors.basePackage("com.xxx.xxx.user.action")))
    .build();
}

```

> 源码分析（多路径自定义扫描控制）

```
public class ApiSelectorBuilder {
  ...
public ApiSelectorBuilder apis(Predicate<RequestHandler> selector) {
     this.requestHandlerSelector = Predicates.and(this.requestHandlerSelector, selector);
     return this;
 }

 public ApiSelectorBuilder paths(Predicate<String> selector) {
     this.pathSelector = Predicates.and(this.pathSelector, selector);
     return this;
 }
...}
```

- package 扫描控制

```
xxx.apis(Predicates.or(new Predicate[]{RequestHandlerSelectors.basePackage("com.xxx.xxx.privilege.api"), RequestHandlerSelectors.basePackage("com.xxx.xxx.user.api"), RequestHandlerSelectors.basePackage("com.xxx.xxx.userauth.api")}))
```
- paths 扫描控制

```
/**正则*/
xx.paths(PathSelectors.ant("/api/v**/**"))
/**多正则匹配*/
xx.paths(pathsRex());
...
private Predicate<String> pathsRex() {
    return or(
            regex("/user.*"),
            regex("/api.*")
    );
}
/**多正则匹配[简写]*/
xx.paths(Predicates.or(new Predicate[]{PathSelectors.regex("/api/v1/**"),PathSelectors.regex("/api/v2/**")}))
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
    RespMapJson test(@RequestParam("userId") int userId);
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


## 打包时剔除springfox-swagger相关jar包

针对于利用swagger.json生成接口pdf和html,有类似于EasyMock的接口管理平台。

> maven

- `parent-pom`

```

<!-- swagger begin -->
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger2</artifactId>
    <version>2.8.0</version>
    <scope>provided</scope>
</dependency>
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger-ui</artifactId>
    <version>2.8.0</version>
    <scope>provided</scope>
</dependency>
<dependency>
    <groupId>io.swagger</groupId>
    <artifactId>swagger-models</artifactId>
    <version>1.5.14</version>
</dependency>
<!-- swagger end -->

```

- `module-pom`

```

 <parent>
        <artifactId>demo</artifactId>
        <groupId>com.example.demo</groupId>
        <version>1.0.0-SNAPSHOT</version>
 </parent>

 ...

<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger2</artifactId>
</dependency>

<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger-ui</artifactId>
</dependency>

<dependency>
    <groupId>io.swagger</groupId>
    <artifactId>swagger-models</artifactId>
</dependency>

```

在parent pom中，springfox的scope设置为provided，swagger的依赖是需要打进运行包的。Springfox以及其依赖的jar都不会打进包中。

    1.provided是没有传递性的，也就是说，如果你依赖的某个jar包，它的某个jar的范围是provided，那么该jar不会在你的工程中依靠jar依赖传递加入到你的工程中。
    2.provided具有继承性，上面的情况，如果需要统一配置一个组织的通用的provided依赖，可以使用parent，然后在所有工程中继承
    3.maven的scope决定依赖的包是否加入本工程的classpath下


| 依赖范围(Scope)	| 编译classpath	| 测试classpath | 运行时classpath | 传递性	|
| :-------- | :----: | :--: | :----:| :--: | :----:|
|compile	| Y	| Y	| Y	| Y	|
|test	    | -	| Y	| - | - |
|provided   | Y	| Y	| - | -	|
|runtime    | - | Y | Y	| Y	|
|system	    | Y | Y | -	| Y	|


<font color="red">关于swagger.json的编译期生成方案：2018-06-27-maven-surefire-plugin.md</font>    

## 文档生成

> 利用asciidoctor插件生成 html 和 pdf 接口文档

[文档生成方式和asciidoctorj生成的pdf文件中文显示不全问题解决方案](https://blog.csdn.net/qq_25215821/article/details/79175535)

处理思路：

- 利用winRAR打开jar包并替换修改后文件
- 下载支持中文的字体文件
- 修改字体配置
- 添加字体xx.ttf文件
- 替换maven或gradle的jar包

`字体目录` asciidoctorj-pdf-1.5.0-alpha.10.1.jar\gems\asciidoctor-pdf-1.5.0.alpha.10\data\fonts

`字体配置文件` asciidoctorj-pdf-1.5.0-alpha.10.1.jar\gems\asciidoctor-pdf-1.5.0.alpha.10\data\themes

[支持中文的asciidoctor-pdf-1.5.0.alpha.10.jar](https://drive.google.com/open?id=1xif0DZkx5ofyWf-Ky2YmBVbBe4GW1toc)

> SpringBoot Gradle搭建文件生成插件

`gradle插件配置`

```
buildscript {
    ext {
        springBootVersion = '1.5.4.RELEASE'
    }
    repositories {
        maven { url 'http://maven.aliyun.com/nexus/content/groups/public/' }
        jcenter()
    }
    dependencies {
        classpath 'org.asciidoctor:asciidoctor-gradle-plugin:1.5.3'
        classpath 'org.asciidoctor:asciidoctorj-pdf:1.5.0-alpha.10.1'
        classpath("org.springframework.boot:spring-boot-gradle-plugin:${springBootVersion}")
        classpath 'io.github.swagger2markup:swagger2markup-spring-restdocs-ext:1.2.0'
        classpath 'io.github.swagger2markup:swagger2markup-gradle-plugin:1.2.0'
        classpath "org.ajoberstar:gradle-git:1.5.1"
    }
}

apply plugin: 'java'
apply plugin: 'maven'
apply plugin: 'idea'
apply plugin: 'spring-boot'
apply plugin: 'io.spring.dependency-management'
apply plugin: 'io.github.swagger2markup'
apply plugin: 'org.asciidoctor.convert'
apply plugin: 'org.ajoberstar.github-pages'

group = 'com.example'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = 1.8

repositories {
    jcenter()
    mavenCentral()
    maven { url 'https://repo.spring.io/snapshot' }
    maven { url 'http://oss.jfrog.org/artifactory/oss-snapshot-local/' }
    mavenLocal()
}

ext {
    restPetsVersion="v1"
    restUserVersion="v1"
    asciiDocOutputDir = file("${buildDir}/asciidoc/generated")
    swaggerOutputDir = file("${buildDir}/swagger")
    swaggerJsonUris = "/v2/api-docs?group=PETS,/v2/api-docs?group=USER"
    swaggerJsonOutputNames = "swagger-pets-${restUserVersion}.json,swagger-user-${restPetsVersion}.json"
    snippetsOutputDir = file("${buildDir}/asciidoc/snippets")
    springfoxVersion = '2.5.0'
}

dependencies {
    compileOnly('org.projectlombok:lombok')
    compile('org.springframework.boot:spring-boot-starter-web')
    compile 'org.springframework.boot:spring-boot-starter-actuator'
    compile 'io.swagger:swagger-annotations:1.5.6'
    compile 'com.google.guava:guava:18.0'
    compile 'net.logstash.logback:logstash-logback-encoder:4.5.1'
    compile("com.fasterxml.jackson.dataformat:jackson-dataformat-smile:2.6.5")
    compile("com.fasterxml.jackson.module:jackson-module-afterburner:2.6.5")

    testCompile "io.springfox:springfox-swagger2:${springfoxVersion}"
    testCompile "io.springfox:springfox-bean-validators:${springfoxVersion}"
    testCompile 'org.springframework.boot:spring-boot-starter-test'
    testCompile 'junit:junit'
    testCompile 'org.springframework.restdocs:spring-restdocs-mockmvc'
    testCompile 'com.fasterxml.jackson.module:jackson-module-jsonSchema:2.6.5'
}



test {
    systemProperty 'rest.user.version', "v1"
    systemProperty 'rest.pets.version', "v1"
    systemProperty 'io.springfox.staticdocs.outputDir', swaggerOutputDir
    systemProperty 'io.springfox.staticdocs.snippetsOutputDir', snippetsOutputDir
    systemProperty 'io.swagger.json.uris', swaggerJsonUris
    systemProperty 'io.swagger.json.output.name', swaggerJsonOutputNames
}

task showProperties << {
    println asciiDocOutputDir
    println swaggerOutputDir
}


convertSwagger2markup {
    dependsOn test
    swaggerInput "${swaggerOutputDir}/swagger.json"
    outputDir asciiDocOutputDir
    config = [
            'swagger2markup.pathsGroupedBy' : 'TAGS',
            'swagger2markup.extensions.springRestDocs.snippetBaseUri': snippetsOutputDir.getAbsolutePath()]
}

asciidoctor {
    dependsOn convertSwagger2markup
    sources {
        include 'index.adoc'
    }
    backends = ['html5', 'pdf']
    attributes = [
            doctype: 'book',
            toc: 'left',
            toclevels: '3',
            numbered: '',
            sectlinks: '',
            sectanchors: '',
            hardbreaks: '',
            generated: asciiDocOutputDir
    ]
}

jar {
    dependsOn asciidoctor
    from ("${asciidoctor.outputDir}/html5") {
        into 'static/docs'
    }
    from ("${asciidoctor.outputDir}/pdf") {
        into 'static/docs'
    }
}

```

`SwaggerConfig`:

```
package com.example.chapter2.config;

import com.google.common.base.Predicates;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Import;
import springfox.bean.validators.configuration.BeanValidatorPluginsConfiguration;
import springfox.documentation.builders.ApiInfoBuilder;
import springfox.documentation.builders.ParameterBuilder;
import springfox.documentation.builders.PathSelectors;
import springfox.documentation.schema.ModelRef;
import springfox.documentation.service.*;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spring.web.plugins.Docket;
import springfox.documentation.swagger2.annotations.EnableSwagger2;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

import static java.util.Arrays.asList;
import static springfox.documentation.builders.PathSelectors.ant;

/**
 * <p>
 *
 * </p>
 *
 * @author xiachaoyang 2018年06月28日 14:42
 * @version V1.0
 * @modificationHistory=========================逻辑或功能性重大变更记录
 * @modify by user: {修改人} 2018年06月28日
 * @modify by reason:{方法名}:{原因}
 */
@EnableSwagger2
@Configuration
@Import(BeanValidatorPluginsConfiguration.class)
public class SwaggerConfig {
    @Bean
    public Docket petApi() {
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(petApiInfo())
                .groupName("PETS")
                .securitySchemes(asList(
                        new OAuth(
                                "petstore_auth",
                                asList(new AuthorizationScope("write_pets", "modify pets in your account"),
                                        new AuthorizationScope("read_pets", "read your pets")),
                                Arrays.<GrantType>asList(new ImplicitGrant(new LoginEndpoint("http://petstore.swagger.io/api/oauth/dialog"), "tokenName"))
                        ),
                        new ApiKey("api_key", "api_key", "header")
                ))
                .select()
                .paths(Predicates.and(ant("/pets/**"), Predicates.not(ant("/error")), Predicates.not(ant("/management/**")), Predicates.not(ant("/management*"))))
                .build();
    }

    @Bean
    public Docket userDocket() {
        return new Docket(DocumentationType.SWAGGER_2)
                .globalOperationParameters(setHeaderToken())
                .apiInfo(userRestInfo())
                .groupName("USER")
                .select()
                .paths(PathSelectors.ant("/user/**"))
                .build();
    }



    @Bean
    public Docket defaultDocket() {
        return new Docket(DocumentationType.SWAGGER_2)
                .globalOperationParameters(setHeaderToken())
                .apiInfo(defaultRestInfo())
                .groupName("DEFAULT")
                .select()
                .paths(PathSelectors.any())
                .build();
    }


    private ApiInfo defaultRestInfo() {
        return new ApiInfoBuilder()
                .title("Swagger DEFAULT")
                .description("REST API Description")
                .contact(new Contact("TestName", "http:/test-url.com", "test@test.de"))
                .license("Apache 2.0")
                .licenseUrl("http://www.apache.org/licenses/LICENSE-2.0.html")
                .version("1.0.0")
                .build();
    }


    private List<Parameter> setHeaderToken() {
        ParameterBuilder token = new ParameterBuilder();
        List<Parameter> parameterList = new ArrayList<>();
        token.name("Token")
                .description("校验 token")
                .modelRef(new ModelRef("String"))
                .parameterType("header")
                .required(true)
                .defaultValue("token1234");
        parameterList.add(token.build());
        return parameterList;
    }

    private ApiInfo userRestInfo() {
        return new ApiInfoBuilder()
                .title("Swagger User")
                .description("User API Description")
                .contact(new Contact("TestName", "http:/test-url.com", "test@test.de"))
                .license("Apache 2.0")
                .licenseUrl("http://www.apache.org/licenses/LICENSE-2.0.html")
                .version("1.0.0")
                .build();
    }

    private ApiInfo petApiInfo() {
        return new ApiInfoBuilder()
                .title("Swagger Petstore")
                .description("Petstore API Description")
                .contact(new Contact("TestName", "http:/test-url.com", "test@test.de"))
                .license("Apache 2.0")
                .licenseUrl("http://www.apache.org/licenses/LICENSE-2.0.html")
                .version("1.0.0")
                .build();
    }
}

```


## 前端对接

> 接口共享

- 提供swagger.json 文档,利用`EasyMock`手动上传接口文件内实现REST-API管理

- swagger文档通过test测试用例生成(打包时不跳过生成swagger的测试用例)

- jenkins自动部署，定时拉取svn/git仓库的项目代码，通过脚本代码执行mvn命令生成swagger文件，放到指定目录

- 开发接口文件解析服务，分析接口变更并通知对应接口关注人。

## REFERENCE

> springfox

[springfox使用手册](https://springfox.github.io/springfox/docs/snapshot/#getting-started)

[springfox-swagger整合教程](http://www.baeldung.com/swagger-2-documentation-for-spring-rest-api)

[SpringMVC集成springfox-swagger2自动生成接口文档](https://www.cnblogs.com/zhaojiankai/p/8318359.html)



```
    https://blog.csdn.net/qq_16256793/article/details/79522749
```


## 更多

> 扫码关注“架构探险之道”，回复`文章名称`获取更多源码和文章资源

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190403222309957.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzI4NjkwNDE3,size_16,color_FFFFFF,t_70)

> 知识星球(扫码加入获取源码和文章资源链接)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190403222322267.jpeg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzI4NjkwNDE3,size_16,color_FFFFFF,t_70)
