---
title: Springfox Swagger 项目接口自动化管理平台
date: 2018-06-08 21:22:00
categories:
- Spring Components
tags:
- spring
- springfox-swagger
---

Springfox Swagger 和Spring的整合已经让我们可以动态的生成接口文档了，但是接口文档和管理如何实现自动化，却也是个让人头疼的问题，下面以公司实战案例作为介绍，希望可以帮助有需要的朋友。    

---
# Springfox Swagger 项目接口自动化管理平台

> 基于公司项目实战的技术总结和可行性方案分析

## 接口文档自动化管理方案

### 编译期生成swagger.json模式

	接口打包忽略springfox依赖

> 获取swagger.json的方式有两种，一种是直接运行组件，在线访问获取文件。另一种是编译期通过mock服务从接口中获取到swagger.json文档。
通过编译生成此文件能够最大化的降低获取文件与组件的运行态依赖，以及能够减少组件不必要的jar包引入

- swagger-pom

swagger相关maven文件放在公共父层,在parent-pom中，springfox的scope设置为provided,Springfox以及其依赖的jar都不会打进war包中

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

- base-pom[项目内最外层pom]

```
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

- SwaggerTest测试类

>为了能在编译期生成文件需要增加一个单元测试类来访问Mock出来的组件服务以获取swagger.json文件

```
@RunWith(SpringJUnit4ClassRunner.class)
@WebAppConfiguration
@ContextConfiguration(locations = { "classpath:*-test.xml"})
```

- maven-surefire-plugin

>为了能在编译时运行测试类需要增加此插件。该插件配置有文件存放的目录<io.springfox.staticdocs.outputDir>，以及swagger.json访问的地址<io.swagger.json.uris>，以及每个文件的命名<io.swagger.json.output.name>,而${project.build.directory}是pom内置属性,默认是/target。配置的swagger.json地址按接口分组来填写，分组几个地址就填写几个，文件名称也是如此

```
# web-pom

<properties>
    ...
    <rest.ui.version>v1</rest.ui.version>
    <rest.api.version>v1</rest.api.version>
</properties>

...

<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-surefire-plugin</artifactId>
    <configuration>
        <systemPropertyVariables>
            <io.springfox.staticdocs.outputDir>${project.build.directory}/swagger</io.springfox.staticdocs.outputDir>
            <io.swagger.json.uris>/v2/api-docs?group=UI,/v2/api-docs?group=API</io.swagger.json.uris>
            <io.swagger.json.output.name>xxx-ui-${rest.ui.version}.json,xxx-api-${rest.api.version}.json</io.swagger.json.output.name>
        </systemPropertyVariables>
    </configuration>
</plugin>

```
<font color="red">如果组件本身的parent-pom设置了此插件并设置了<skip>true</skip>则测试类不会运行，得将此设置去除</font>

### 利用MAVEN生成swagger.json

> 思路

- Test中利用Mock生成swagger.json
- 利用maven-surefire-plugin插件执行
- 指定执行SwaggerTest.java
- svn提交代码，jenknis框架通过脚本判断svn代码是否有更新，有更新则拉取副本，执行机通过bat[windows服务器]执行mvn clean install 命令生成文件到项目target/swagger下
- 创建服务定时拉取swagger.json文件
- 搭建EasyMock平台，自动提交并生成接口文档
- 每次变更向接口关注人发送邮件推送接口变更消息

> Springfox Swagger配置

- 打包时跨过springfox相关依赖
- 执行测试类生成swagger.json

<font color="red">[配置方案]</font>

#### Mock数据源注入

```
package com.xxx.controller.common.config;

import com.alibaba.druid.pool.DruidDataSource;
import com.xxx.dolphin.resources.synchronize.log.LocalFileLogger;
import org.easymock.EasyMock;
import org.easymock.IMocksControl;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.jdbc.core.namedparam.NamedParameterJdbcTemplate;

import javax.sql.DataSource;

/**
 * <p>
 * mock注入SwaggerTest执行所需实例
 * </p>
 *
 * @author xiachaoyang 2018年06月26日 14:03
 * @version V1.0
 * @modificationHistory=========================逻辑或功能性重大变更记录
 * @modify by user: {修改人} 2018年06月26日
 * @modify by reason:{方法名}:{原因}
 */
public class BeanMockFactory {

    //-- 数据源相关

    public DataSource mockDataSource() {
        IMocksControl control = EasyMock.createControl();
        DruidDataSource bean = control.createMock(DruidDataSource.class);
        return bean;
    }

    public JdbcTemplate mockJdbcTemplate() {
        IMocksControl control = EasyMock.createControl();
        JdbcTemplate bean = control.createMock(JdbcTemplate.class);
        return bean;
    }

    public NamedParameterJdbcTemplate mockNamedParameterJdbcTemplate() {
        IMocksControl control = EasyMock.createControl();
        NamedParameterJdbcTemplate bean = control.createMock(NamedParameterJdbcTemplate.class);
        return bean;
    }

    //-- 日志相关

    public LocalFileLogger mockLocalFileLogger() {
        IMocksControl control = EasyMock.createControl();
        LocalFileLogger bean = control.createMock(LocalFileLogger.class);
        return bean;
    }

    public xxxBusinessLogDao mockxxxBusinessLogDao(){
        IMocksControl control = EasyMock.createControl();
        xxxBusinessLogDao bean = control.createMock(xxxBusinessLogDao.class);
        return bean;
    }
}

```

#### SwaggerTest类中配置扫描


```
@RunWith(SpringJUnit4ClassRunner.class)
@WebAppConfiguration
@ContextConfiguration(locations = { "classpath:*-test.xml"})
public class SwaggerTest {
	...
}

```

- xml中通过mock实现虚拟实例注入


```
<bean id = "beanMockFactory" class ="com.xxx.web.config.BeanMockFactory"/>

<bean id="dataSource" factory-method="mockDataSource" factory-bean="beanMockFactory"/>

<bean id="jdbcTemplate" factory-method="mockJdbcTemplate" factory-bean="beanMockFactory"/>

<bean id="namedJdbcTemplate" factory-method="mockNamedParameterJdbcTemplate" factory-bean="beanMockFactory"/>

<bean id="xxxBusinessLogDao" factory-method="mockxxxBusinessLogDao" factory-bean="beanMockFactory"/>
```

- 调整xml文件名，确保以-test.xml后缀，并将`*-test.xml`相关的配置xml直接放在test的resource下
- 处理mybatis相关的注入[主要是数据源Mock和Mapper的扫描路径检查]
- 检查xml配置文件间通过import引入的xml是否都是以-test后缀结尾的文件，避免误引
- 读取本地配置文件后完成的实例注入需要利用BeanMockFactory中完成注入
- 通过JdbcTemplet自定义完成Dao数据层查询的示例需要通过Mock注入,因为myabtis的扫描不会处理自定义Dao类
- 静态方法调用(读取配置文件中的变量),需要添加try...catch...异常处理,捕获异常但是不抛出
- service层未调用，而是通过xml中扫描完成的实例注入可以直接注释或删除
- AMQ相关的代码中启动未连接则不需要Mock

`任务示例`

```

# 任务相关的示例类扫描不要注释，只注释任务的执行计划(如下)，避免任务执行

<task:scheduler id="xxx_task" pool-size="4"/>

<task:scheduled-tasks scheduler="xxx_task">
   ...
</task:scheduled-tasks>

```

`线程、监听器等示例`


```
<context:component-scan base-package="com.xxx.notify"/>
<context:component-scan base-package="com.xxx.*.client"/>
<context:component-scan base-package="com.xxx.*.service"/>
<context:component-scan base-package="com.xxx.*.listenter"/>
<context:component-scan base-package="com.xxx.*.thread"/>

```

	只要不是扫描后启动则不需要去注释和Mock，正常扫描即可

`LDAP、resource示例`

	正常扫描，除示例化需要读取配置文件外，其他都不需要处理，读取本地配置文件的需要手动在BeanMockFactory中添加实例mock方法


`maven-surefire-plugin`

	api、core、web层中使用该插件不要配置	<skipTest>true</skipTest> (跳过执行测试用例)


```xml

<plugin>  
    <groupId>org.apache.maven.plugins</groupId>  
    <artifactId>maven-surefire-plugin</artifactId>  
    <version>2.5</version>  
    <configuration>  
        <skipTests>true</skipTests>  
    </configuration>  
</plugin>

```

- web-pom-config

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-surefire-plugin</artifactId>
    <configuration>
        <systemPropertyVariables>
            <io.springfox.staticdocs.outputDir>${project.build.directory}/swagger</io.springfox.staticdocs.outputDir>
            <io.swagger.json.uris>/v2/api-docs?group=UI,/v2/api-docs?group=API</io.swagger.json.uris>
            <io.swagger.json.output.name>xxx-ui-${rest.ui.version}.json,xxx-api-${rest.api.version}.json</io.swagger.json.output.name>
        </systemPropertyVariables>
    </configuration>
</plugin>
```

- parent-pom-config


```xml

<plugin>
	<groupId>org.apache.maven.plugins</groupId>
	<artifactId>maven-surefire-plugin</artifactId>
	<configuration>
		<includes>
			<include>**/SwaggerTest.java</include>
		</includes>
	</configuration>
</plugin>

```

- 测试日志

```

# IDAE MAVEN
mvn clean install

# cmd
# 不执行测试用例，但编译测试用例类生成相应的class文件至target/test-classes下。
mvn clean install -Dmaven.test.skip=false
相当于
<plugin>  
    <groupId>org.apache.maven.plugins</groupId>  
    <artifactId>maven-surefire-plugin</artifactId>  
    <version>2.5</version>  
    <configuration>  
        <skip>true</skip>  
    </configuration>  
</plugin>

# log
target/surefire-reports目录下
com.xxx.web.test.SwaggerTest.txt
阅读日志并解决相关报错，其他无需关注

# 正常生成打印日志如下
-------------------------------------------------------------------------------
Test set: com.xxx.web.test.SwaggerTest
-------------------------------------------------------------------------------
Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 11.735 sec

```

<font color="red">[核心]</font>

	- controller和相关数据Dto扫描
	- controller中依赖的servcie、task等实例注入类引用（Autowired、Resource等）
	- 观察测试日志，解决影响swagger.json生成的报错


#### 配置遇到的问题

-  maven-surefire-plugin插件中skip和skipTests区别

	../../env/maven/2018-06-27-maven-surefire-plugin.md

- 生成的文档里只有一个license的报错返回日志

```

{"code":"0x02f10003","msg":"xxx.xxx.license.expire","data":null}

```

	将License相关的拦截器配置注释

```
<!-- 拦截器 配置多个将会顺序执行 -->
<mvc:interceptors>
	<bean class="com.xxx.common.web.interceptor.HttpServletResponseInterceptor" />
	<bean class="com.xxx.common.web.interceptor.NoCacheInterceptor"/>
<!--<bean class="com.xxx.interceptor.LicenseVaildateInterceptor" />-->
</mvc:interceptors>
```

- 无法找到resource下文件夹包裹的xml配偶文件

	若非直接放入resource下，而是含有`resource/xml/*-test-xml`中的xml之类的文件夹，实际执行中会报中间目录找不到的问题

- 生成swagger的测试类依赖的servlet jar包2.5和3.0版本冲突


```

# 报错信息

java.lang.NoSuchMethodError: javax.servlet.http.HttpServletRequest.getAsyncContext()Ljavax/servlet/AsyncContext;

	at org.springframework.test.web.servlet.TestDispatcherServlet.initAsyncDispatchLatch(TestDispatcherServlet.java:88)
	at org.springframework.test.web.servlet.TestDispatcherServlet.service(TestDispatcherServlet.java:68)
	at javax.servlet.http.HttpServlet.service(HttpServlet.java:820)
	at org.springframework.mock.web.MockFilterChain$ServletFilterProxy.doFilter(MockFilterChain.java:160)
	at org.springframework.mock.web.MockFilterChain.doFilter(MockFilterChain.java:127)
	at org.springframework.test.web.servlet.MockMvc.perform(MockMvc.java:151)
	at com.xxx.controller.common.test.SwaggerTest.getSwaggerJson(SwaggerTest.java:75)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at org.junit.runners.model.FrameworkMethod$1.runReflectiveCall(FrameworkMethod.java:50)
	at org.junit.internal.runners.model.ReflectiveCallable.run(ReflectiveCallable.java:12)
	at org.junit.runners.model.FrameworkMethod.invokeExplosively(FrameworkMethod.java:47)
	at org.junit.internal.runners.statements.InvokeMethod.evaluate(InvokeMethod.java:17)
	at org.junit.internal.runners.statements.RunBefores.evaluate(RunBefores.java:26)
	at org.springframework.test.context.junit4.statements.RunBeforeTestMethodCallbacks.evaluate(RunBeforeTestMethodCallbacks.java:75)
	at org.springframework.test.context.junit4.statements.RunAfterTestMethodCallbacks.evaluate(RunAfterTestMethodCallbacks.java:86)
	at org.springframework.test.context.junit4.statements.SpringRepeat.evaluate(SpringRepeat.java:84)
	at org.junit.runners.ParentRunner.runLeaf(ParentRunner.java:325)
	at org.springframework.test.context.junit4.SpringJUnit4ClassRunner.runChild(SpringJUnit4ClassRunner.java:252)
	at org.springframework.test.context.junit4.SpringJUnit4ClassRunner.runChild(SpringJUnit4ClassRunner.java:94)
	at org.junit.runners.ParentRunner$3.run(ParentRunner.java:290)
	at org.junit.runners.ParentRunner$1.schedule(ParentRunner.java:71)
	at org.junit.runners.ParentRunner.runChildren(ParentRunner.java:288)
	at org.junit.runners.ParentRunner.access$000(ParentRunner.java:58)
	at org.junit.runners.ParentRunner$2.evaluate(ParentRunner.java:268)
	at org.springframework.test.context.junit4.statements.RunBeforeTestClassCallbacks.evaluate(RunBeforeTestClassCallbacks.java:61)
	at org.springframework.test.context.junit4.statements.RunAfterTestClassCallbacks.evaluate(RunAfterTestClassCallbacks.java:70)
	at org.junit.runners.ParentRunner.run(ParentRunner.java:363)
	at org.springframework.test.context.junit4.SpringJUnit4ClassRunner.run(SpringJUnit4ClassRunner.java:191)
	at org.junit.runner.JUnitCore.run(JUnitCore.java:137)
	at com.intellij.junit4.JUnit4IdeaTestRunner.startRunnerWithArgs(JUnit4IdeaTestRunner.java:68)
	at com.intellij.rt.execution.junit.IdeaTestRunner$Repeater.startRunnerWithArgs(IdeaTestRunner.java:47)
	at com.intellij.rt.execution.junit.JUnitStarter.prepareStreamsAndStart(JUnitStarter.java:242)
	at com.intellij.rt.execution.junit.JUnitStarter.main(JUnitStarter.java:70)

```

	原因:

		swagger.json生成需要通过mock模拟请求,请求的处理依赖servlet.api的3.0版本的jar包，但是由于maven引用的jar包冲突，虽然项目中存在其他jar包依赖而导入的3.0版本的servlet.api,但项目中配置的基础依赖是基于2.5版本的。实际运行时调用的是2.5版本的。
		可通过在IDEA中的项目lib管理中删除2.5版本的jar包后来执行测试类，实际运行结果是可以生成的，所以核心问题是解决jar包版本冲突。

```
# 核心错误代码

...
import javax.servlet.http.HttpServletRequest;
...

final class TestDispatcherServlet extends DispatcherServlet {

	...

	private void initAsyncDispatchLatch(HttpServletRequest request) {
			if (request.getAsyncContext() != null) {
				final CountDownLatch dispatchLatch = new CountDownLatch(1);
				((MockAsyncContext) request.getAsyncContext()).addDispatchHandler(new Runnable() {
					@Override
					public void run() {
						dispatchLatch.countDown();
					}
				});
				getMvcResult(request).setAsyncDispatchLatch(dispatchLatch);
			}
	}

	...

}

```

	分析：

		2.5版本的servlet.api中request.getAsyncContext()方法未定义，3.0.1版本中含有该方法，所以需要调整pom依赖引用顺序

	解决方式：

		在测试类所在层web-pom中添加3.0的servlet依赖配置，并将顺序调整至2.5前方。


```
# web-pom

<!-- servlet-api版本依赖声明 -->
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>javax.servlet-api</artifactId>
    <scope>provided</scope>
</dependency>

<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>servlet-api</artifactId>
    <scope>provided</scope>
</dependency>

# parent-pom

<properties>
	...
    <servlet-version>2.5</servlet-version>
    <javax.servlet-version>3.1.0</javax.servlet-version>
    ...
 </properties>


<!-- servlet-api版本依赖声明 -->
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>servlet-api</artifactId>
    <version>${servlet-version}</version>
    <scope>provided</scope>
</dependency>

<!-- servlet-api版本依赖声明 -->
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>javax.servlet-api</artifactId>
    <version>${javax.servlet-version}</version>
</dependency>
```


## 微信公众号

<center>
<img src="https://images.gitee.com/uploads/images/2018/0717/215030_8e782063_912956.png" width="50%" height="50%"/>
</center>

扫码关注或搜索`架构探险之道`获取最新文章，不积跬步无以至千里，坚持每周一更，坚持技术分享。我和你们一起成长 ^_^ ！
