---
title: Spring Boot 整合 Activiti 6.0.0 工作流引擎开发
date: 2018-07-21 23:22:00
categories:
- Activiti
tags:
- spring-boot
---

本教程基于Activiti 6.0.0 ，着力介绍工作流引擎Activiti6.0.0引擎和Spring Boot的整合开发，帮助初学者入门。

---

# Spring Boot 整合 Activiti 6.0.0 工作流引擎开发

本教程基于Activiti 6.0.0 ，着力介绍工作流引擎Activiti6.0.0引擎和Spring Boot的整合开发入门教程。

---


## 开发环境
	1. Tomcat 7.0.78
	2. JDK 7+
	3.  Activiti 6.0.0
	4.  spring-boot-starter-parent  1.4.2.RELEASE
	5.  mybatis

## maven

```
<dependency>
    <groupId>org.activiti</groupId>
    <artifactId>activiti-spring-boot-starter-basic</artifactId>
    <version>${activiti.version}</version>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-rest</artifactId>
</dependency>
```

## 多数据源配置

```
package cn.showclear.utio.config;

import cn.showclear.utio.mybatis.DataSources;
import cn.showclear.utio.mybatis.ThreadLocalRountingDataSource;
import org.activiti.spring.SpringAsyncExecutor;
import org.activiti.spring.SpringProcessEngineConfiguration;
import org.activiti.spring.boot.AbstractProcessEngineAutoConfiguration;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.*;
import org.springframework.jdbc.datasource.DriverManagerDataSource;
import org.springframework.transaction.PlatformTransactionManager;

import javax.sql.DataSource;
import java.io.IOException;
import java.util.HashMap;
import java.util.Map;

/**
 * Created by Yiyuery.
 */
@Configuration
@ComponentScan
@PropertySource(value = {"classpath:/application.properties",
        "file:/C:\\scooper\\utio\\db.properties","file:/icooper/config/utio/db.properties"},
        ignoreResourceNotFound = true)
public class DataSourceConfig extends AbstractProcessEngineAutoConfiguration {

    @Bean
    @ConfigurationProperties(prefix="db.other")
    public DataSource dataSourceaOther() {
        return new DriverManagerDataSource();
    }

    @Primary
    @Bean(name = "dataSource")
    public ThreadLocalRountingDataSource dataSource(){
        ThreadLocalRountingDataSource dataSource = new ThreadLocalRountingDataSource();
        dataSource.setDefaultTargetDataSource(dataSourceUtio());
        Map<Object , Object> dataSourceList = new HashMap();
        dataSourceList.put(DataSources.OTHER,dataSourceaOther());        
        dataSource.setTargetDataSources(dataSourceList);
        return dataSource;
    }

	@Bean
    @ConfigurationProperties(prefix = "spring.datasource.activiti")
    public DataSource dataSourceActiviti() {
        return DataSourceBuilder.create().build();
    }

    @Bean
    public SpringProcessEngineConfiguration springProcessEngineConfiguration(
            PlatformTransactionManager transactionManager,
            SpringAsyncExecutor springAsyncExecutor) throws IOException {

        return baseSpringProcessEngineConfiguration(
                dataSourceActiviti(),
                transactionManager,
                springAsyncExecutor);
    }
}

```
## 核心配置文件

```
/*resources/application.properties*/
spring.jpa.hibernate.ddl-auto=update
spring.jpa.database=MYSQL
spring.datasource.activiti.url=jdbc:mysql://192.168.106.104:3306/DB_SC_ACTIVITI?characterEncoding=UTF-8
spring.datasource.activiti.username=showclear
spring.datasource.activiti.password=showclear
spring.datasource.activiti.driver-class-name=com.mysql.jdbc.Driver
```


> SpringBoot 整合开发笔记

## Activiti 简介

    1、类似于OA的一种流式工作任务管理框架。
    2、依赖于Activiti BPM引擎和BPMN 2.0


## 流程设计器的搭建

> 官网 https://www.activiti.org/

#### 下载官方流程设计器
https://www.activiti.org/download-links
> 目前最新的版本是6.0.0

![这里写图片描述](https://img-blog.csdn.net/20180404095804613?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzI4NjkwNDE3/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

####  将解压后的wars放在Tomcat下：
并修改配置文件为mysql的[连接方式](http://note.youdao.com/noteshare?id=25c687dea4a92ec0b5b48302e625cd22&sub=C2EB1907F8B94DE9B34C51388BEA30D1)

    配置文件所在目录：tomcat\apache-tomcat-7.0.41\webapps\activiti-app\WEB-INF\classes\META-INF\activiti-app

 ![这里写图片描述](https://img-blog.csdn.net/20180404095824661?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzI4NjkwNDE3/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)



##  初始化数据库

####  SpringBoot 整合开发

> github

    https://github.com/Activiti/Activiti

> maven
```
<!-- activiti-start -->
	<dependency>
		<groupId>org.activiti</groupId>
		<artifactId>activiti-spring-boot-starter-basic</artifactId>
		<version>${activiti.version}</version>
	</dependency>
	<!-- spring jpa中自带tomcat数据连接池 -->
	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-data-jpa</artifactId>
	</dependency>

	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-data-rest</artifactId>
	</dependency>
<!-- activiti-end -->
```

> 数据源配置和工作流引擎对象创建

```
@Configuration
public class ActivitiConfig  extends AbstractProcessEngineAutoConfiguration {

    /**
     * 工作流数据库
     */
    @Autowired
    @Qualifier("activiti")
    private DataSource activitiDataSource;

    /**
     * 工作流 引擎对象创建
     * @param transactionManager
     * @param springAsyncExecutor
     * @return
     * @throws IOException
     */
    @Bean
    public SpringProcessEngineConfiguration springProcessEngineConfiguration(
            PlatformTransactionManager transactionManager,
            SpringAsyncExecutor springAsyncExecutor) throws IOException {

        return baseSpringProcessEngineConfiguration(
                activitiDataSource,
                transactionManager,
                springAsyncExecutor);
    }
}

/*关于多数据源的配置方式网上有很多种，可以自己了解下*/

```

> Activiti配置文件（用来生成和初始化数据库）

```

/*文件需放在resources下的processes下*/

<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans   http://www.springframework.org/schema/beans/spring-beans.xsd">

 <bean id="processEngineConfiguration" class="org.activiti.engine.impl.cfg.StandaloneProcessEngineConfiguration">	    
	 <!-- 连接数据的配置 -->
    <property name="jdbcDriver" value="com.mysql.jdbc.Driver"></property>
	<property name="jdbcUrl" value="jdbc:mysql://127.0.0.1:3306/DB_SC_ACTIVITI?characterEncoding=utf8&amp;zeroDateTimeBehavior=convertToNull"></property>
	<property name="jdbcUsername" value="showclear"></property>
	<property name="jdbcPassword" value="showclear"></property>   
	<property name="databaseSchemaUpdate" value="true" />
	<property name="history" value="full" />
	<property name="processDefinitionCacheLimit" value="10" />
	<property name="activityFontName" value="宋体"/>
	<property name="labelFontName" value="宋体"/>
  </bean>

</beans>
```

> 生成数据库

```
/**
 * 使用配置文件来创建数据库中的表
 */
@Test
public void createTable() {

    ProcessEngine processEngine = ProcessEngineConfiguration.createProcessEngineConfigurationFromResource("processes/activiti.cfg.xml")
            .buildProcessEngine();
    System.out.println(processEngine);
}
```

> 打开流程设计器

![这里写图片描述](https://img-blog.csdn.net/20180404095835456?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzI4NjkwNDE3/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

> 开发指导文档

    https://www.activiti.org/userguide/6.latest/

## 工作流的部署和任务执行

>  以入廊申请为例

```
/**
 * 入廊申请测试
 * @author YF-XIACHAOYANG
 * @date 2017/8/3 17:13
 */
public class EntryPipeApplication {

    /**
     * RepositoryService是管理流程定义的仓库服务的接口。
     */
    private static RepositoryService repositoryService;
    /**
     * RuntimeService是activiti的流程执行服务类。可以从这个服务类中获取很多关于流程执行相关的信息，如执行管理，包括启动、推进、删除流程实例等操作。
     */
    private static RuntimeService runtimeService;
    /**
     * TaskService是activiti的任务服务类。可以从这个类中获取任务的信息
     */
    private static TaskService taskService;
    /**
     * HistoryService 是activiti的查询历史信息的类。在一个流程执行完成后，这个对象为我们提供查询历史信息。
     */
    private static HistoryService historyService;
    /**
     * 工作流核心引擎对象
     */
    private static ProcessEngine processEngine;

    @Before
    public void init() {
        createProcessEngine();
    }

    private static String BPMN_XML_NAME = "EntryPipeApplication.bpmn20.xml";

    private static String BPMN_XML_DESC = "入廊申请工作流";


    /**
     * 创建一个单例的ProcessEngine
     */
    public void createProcessEngine() {
        processEngine = ProcessEngineConfiguration.createProcessEngineConfigurationFromResource("processes/activiti.cfg.xml")
                .buildProcessEngine();
        System.out.println("Activiti ProcessEngine Build Success!");
        getActivitiService();
    }

    /**
     * 获取activiti相关的service服务
     */
    public void getActivitiService() {
        repositoryService = processEngine.getRepositoryService();
        runtimeService = processEngine.getRuntimeService();
        taskService = processEngine.getTaskService();
        historyService = processEngine.getHistoryService();
    }

    /**
     * 部署流程定义
     */
    @Test
     public void deploymentProcessDefinition() {

        Deployment deployment = repositoryService// 与流程定义和部署对象相关的service
                .createDeployment()// 创建一个部署对象
                .name(BPMN_XML_DESC)// 添加部署的名称
                .addClasspathResource("processes/"+BPMN_XML_NAME)// classpath的资源中加载，一次只能加载
                .deploy();// 完成部署
        System.out.println("部署ID:" + deployment.getId());
        System.out.println("部署名称：" + deployment.getName());
    }

    /**
     * 启动一个入廊申请流程
     */
    @Test
    public void start() {
        Map<String, Object> variables = new HashMap<String, Object>();
        //启动流程参数设置
        String processId = runtimeService.startProcessInstanceByKey("entryPipeApplication", variables).getId();
        System.out.println("***************启动一个入廊申请流程完成***************" + processId);
        //17501
    }

    /**
     * 审核请假申请工作流查询
     */
    @Test
    public void queryRunTaskByTaskId()  {

        List<Task> list = processEngine.getTaskService()// 与正在执行的认为管理相关的Service
                .createTaskQuery()// 创建任务查询对象
                .list();
        if (list != null && list.size() > 0) {
            for (Task task : list) {
                String procId = task.getProcessInstanceId();
                if (procId.equals("17501")) {
                    System.out.println("#----------------------------------------#");
                    System.out.println("任务ID:" + task.getId());
                    System.out.println("任务名称:" + task.getName());
                    System.out.println("任务的创建时间" + task);
                    System.out.println("任务的办理人:" + task.getAssignee());
                    System.out.println("流程实例ID:" + task.getProcessInstanceId());
                    System.out.println("执行对象ID:" + task.getExecutionId());
                    System.out.println("流程定义ID:" + task.getProcessDefinitionId());
                    System.out.println("#----------------------------------------#");
                }
            }
        }

    }

    /**
     * 发起入廊申请[填写表单]
     */
    @Test
    public void startApplyTask(){
        String taskId="17505";  //任务Id
        Map<String, Object> taskVariables = new HashMap<String, Object>();
        taskVariables.put("applyUserId", 123);
        taskVariables.put("utId", 12);
        taskVariables.put("deptUserId", 31);
        //完成
        processEngine.getTaskService()//与正在执行的认为管理相关的Service
                .complete(taskId, taskVariables);
        System.out.println("完成任务:任务ID:"+taskId);
    }


    /**
     * 部门领导审核
     */
    @Test
    public void deptVerifyTask() {
        String taskId="20008";  //任务Id
        Map<String, Object> taskVariables = new HashMap<String, Object>();
        taskVariables.put("applyDeptApproved", 1);
        taskVariables.put("deptOpinion", "通过！");
        taskVariables.put("opcUserId", 32);
        //完成
        processEngine.getTaskService()//与正在执行的认为管理相关的Service
                .complete(taskId, taskVariables);
        System.out.println("完成任务:任务ID:"+taskId);
    }

    /**
     * 运维中心审核
     */
    @Test
    public void opcVerifyTask(){

        String taskId="22509";  //任务Id
        Map<String, Object> taskVariables = new HashMap<String, Object>();
        taskVariables.put("applyOpcApproved", 1);
        taskVariables.put("deptOpinion", "通过！");
        taskVariables.put("adminUserId", 33);
        //完成
        processEngine.getTaskService()//与正在执行的认为管理相关的Service
                .complete(taskId, taskVariables);
        System.out.println("完成任务:任务ID:"+taskId);
    }


    /**
     * 行政部门审核
     */
    @Test
    public void adminVerifyTask(){

        String taskId="25008";  //任务Id
        Map<String, Object> taskVariables = new HashMap<String, Object>();
        taskVariables.put("applyAdminApproved", 1);
        taskVariables.put("deptOpinion", "通过！");

        //完成
        processEngine.getTaskService()//与正在执行的认为管理相关的Service
                .complete(taskId, taskVariables);
        System.out.println("完成任务:任务ID:"+taskId);
    }

/*
    任务ID:2505
    任务名称:发起入廊申请
    任务的创建时间Task[id=2505, name=发起入廊申请]
    任务的办理人:null
    流程实例ID:2501
    执行对象ID:2502
    流程定义ID:entryPipeApplication:1:4

    完成任务:任务ID:2505

    再次查询：
    任务ID:5006
    任务名称:部门领导审核
    任务的创建时间Task[id=5006, name=部门领导审核]
    任务的办理人:null
    流程实例ID:2501
    执行对象ID:2502
    流程定义ID:entryPipeApplication:1:4

    任务ID:7505
    任务名称:发起入廊申请
    任务的创建时间Task[id=7505, name=发起入廊申请]
    任务的办理人:null
    流程实例ID:2501
    执行对象ID:2502
    流程定义ID:entryPipeApplication:1:4


    完成任务:任务ID:7505

    任务ID:10006
    任务名称:部门领导审核
    任务的创建时间Task[id=10006, name=部门领导审核]
    任务的办理人:null
    流程实例ID:2501
    执行对象ID:2502
    流程定义ID:entryPipeApplication:1:4


    完成任务:任务ID:10006

    任务ID:12505
    任务名称:运维中心审核
    任务的创建时间Task[id=12505, name=运维中心审核]
    任务的办理人:null
    流程实例ID:2501
    执行对象ID:2502
    流程定义ID:entryPipeApplication:1:4
#-----------------------------------
 */


}

```


## 国内比较成熟的工作流框架

> Lemon OA

    http://www.mossle.com/index.do

## 微信公众号

<center>
<img src="https://images.gitee.com/uploads/images/2018/0717/215030_8e782063_912956.png" width="50%" height="50%"/>
</center>

扫码关注或搜索`架构探险之道`获取最新文章，不积跬步无以至千里，坚持每周一更，坚持技术分享。我和你们一起成长 ^_^ ！
