---
title: SpringMVC 整合 Mybatis 开发笔记
date: 2018-04-04 11:16:00
categories:
- DataBase
tags:
- mybatis
---

MyBatis 是一款优秀的持久层框架，它支持定制化 SQL、存储过程以及高级映射。MyBatis 避免了几乎所有的 JDBC 代码和手动设置参数以及获取结果集。MyBatis 可以使用简单的 XML 或注解来配置和映射原生信息，将接口和 Java 的 POJOs(Plain Old Java Objects,普通的 Java对象)映射成数据库中的记录。

---

#  SpringMVC 整合 Mybatis 开发笔记

## maven

```
 <!-- mybatis ORM spring集成框架 -->
  <dependency>
      <groupId>org.mybatis</groupId>
      <artifactId>mybatis-spring</artifactId>
      <version>1.3.0</version>
  </dependency>
```
## mybatis配置和数据库配置

> sqlMapConfig.xml （含分页插件)

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN" "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <settings>
        <setting name="callSettersOnNulls" value="true"/>
        <!-- 这个配置使全局的映射器启用或禁用缓存 -->
        <setting name="cacheEnabled" value="true" />
        <!-- 允许 JDBC 支持生成的键。需要适合的驱动。如果设置为 true 则这个设置强制生成的键被使用，尽管一些驱动拒绝兼容但仍然有效（比如 Derby） -->
        <setting name="useGeneratedKeys" value="true" />
        <!-- 配置默认的执行器。SIMPLE 执行器没有什么特别之处。REUSE 执行器重用预处理语句。BATCH 执行器重用语句和批量更新  -->
        <setting name="defaultExecutorType" value="REUSE" />
        <!-- 全局启用或禁用延迟加载。当禁用时，所有关联对象都会即时加载。默认：true  -->
        <!-- <setting name="lazyLoadingEnabled" value="true"/>  -->
        <!-- 当启用时，有延迟加载属性的对象在被调用时将会完全加载任意属性 . 默认：true-->
        <!-- <setting name="aggressiveLazyLoading" value="true"/>   -->
        <!-- 设置超时时间，它决定驱动等待一个数据库响应的时间。  -->
        <setting name="defaultStatementTimeout" value="25000"/>
    </settings>

    <plugins>
        <!-- com.github.pagehelper为PageHelper类所在包名 -->
        <plugin interceptor="com.github.pagehelper.PageHelper">
            <property name="dialect" value="mysql"/>
            <!-- 该参数默认为false -->
            <!-- 设置为true时，会将RowBounds第一个参数offset当成pageNum页码使用 -->
            <!-- 和startPage中的pageNum效果一样-->
            <property name="offsetAsPageNum" value="true"/>
            <!-- 该参数默认为false -->
            <!-- 设置为true时，使用RowBounds分页会进行count查询 -->
            <property name="rowBoundsWithCount" value="true"/>
        </plugin>
    </plugins>
</configuration>

```

> spring-dao.xml  数据配置

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/tx
       http://www.springframework.org/schema/tx/spring-tx.xsd">

    <description>Spring-Database配置</description>

    <!-- C3P0 数据源配置 -->
    <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource" destroy-method="close">
        <property name="driverClass" value="${db.activiti.driver}"/>
        <property name="jdbcUrl" value="${db.activiti.url}"/>
        <property name="user" value="${db.activiti.username}"/>
        <property name="password" value="${db.activiti.password}"/>
        <!--初始化时获取的连接数，取值应在minPoolSize与maxPoolSize之间。Default: 3 -->
        <property name="initialPoolSize" value="${db.activiti.initialSize}"></property>
        <!--连接池中保留的最大连接数。Default: 15 -->
        <property name="maxPoolSize" value="${db.activiti.maxActive}"></property>
        <!--最大空闲时间,60秒内未使用则连接被丢弃。若为0则永不丢弃。Default: 0 -->
        <property name="maxIdleTime"> <value>60</value> </property>
        <!--当连接池中的连接耗尽的时候c3p0一次同时获取的连接数。Default: 3 -->
        <property name="acquireIncrement"><value>5</value></property>
        <!--JDBC的标准参数，用以控制数据源内加载的PreparedStatements数量。但由于预缓存的statements
         属于单个connection而不是整个连接池。所以设置这个参数需要考虑到多方面的因素。
         如果maxStatements与maxStatementsPerConnection均为0，则缓存被关闭。Default: 0-->
        <property name="maxStatements"><value>0</value></property>
        <!--每60秒检查所有连接池中的空闲连接。Default: 0 -->
        <property name="idleConnectionTestPeriod"><value>60</value></property>
        <!--定义在从数据库获取新连接失败后重复尝试的次数。Default: 30 -->
        <property name="acquireRetryAttempts"><value>30</value></property>
        <!--获取连接失败将会引起所有等待连接池来获取连接的线程抛出异常。但是数据源仍有效
         保留，并在下次调用getConnection()的时候继续尝试获取连接。如果设为true，那么在尝试
         获取连接失败后该数据源将申明已断开并永久关闭。Default: false-->
        <property name="breakAfterAcquireFailure"><value>true</value></property>
        <!--因性能消耗大请只在需要的时候使用它。如果设为true那么在每个connection提交的
         时候都将校验其有效性。建议使用idleConnectionTestPeriod或automaticTestTable
         等方法来提升连接测试的性能。Default: false -->
        <property name="testConnectionOnCheckout"><value>false</value></property>
    </bean>

    <!-- MyBatis配置 -->
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="dataSource" />
        <property name="configLocation" value="classpath:/config/sqlMapConfig.xml" />
        <!-- 自动扫描entity目录, 省掉Configuration.xml里的手工配置 -->
      <!--  <property name="typeAliasesPackage" value="cn.com.showclear.activiti.pojo.activiti" />-->
        <!-- 显式指定Mapper文件位置 -->
        <property name="mapperLocations" value="classpath:/config/mappers/activiti/*.xml" />
    </bean>

    <!-- DAO接口所在包名，Spring会自动查找其下的类 -->
    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <property name="basePackage" value="cn.com.showclear.activiti.dao" />
        <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"></property>
    </bean>

    <!--事务配置-->
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"/>
    </bean>
    <tx:annotation-driven transaction-manager="transactionManager" mode="proxy" proxy-target-class="true"/>
    <tx:advice id="txAdvice" transaction-manager="transactionManager">
        <tx:attributes>
            <tx:method name="find*" read-only="true" />
            <tx:method name="get*" read-only="true" />
            <tx:method name="page*" read-only="true" />
            <tx:method name="query*" read-only="true" />
            <tx:method name="unique*" read-only="true" />
            <tx:method name="*" read-only="false" />
            <tx:method name="*" rollback-for="Exception" />
        </tx:attributes>
    </tx:advice>
    <tx:annotation-driven transaction-manager="transactionManager" />

</beans>
```

## generatorConfig.xml

> pojo、mapper.xml、dao 逆向生成插件

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
        PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">
<generatorConfiguration>
    <!--数据库驱动-->
    <classPathEntry location="F:\sync\tools\jar\mysql\mysql-connector-java-5.1.40.jar"/>
    <!--  flat 模式不会产生blobs类-->
    <context id="scooper" defaultModelType="flat" targetRuntime="MyBatis3">
        <commentGenerator>
            <property name="javaFileEncoding" value="UTF-8"/>
            <property name="suppressDate" value="false"/>
            <property name="suppressAllComments" value="true"/>
        </commentGenerator>

        <!--物资管理-->
        <!-- 数据库链接地址账号密码-->
        <jdbcConnection driverClass="com.mysql.jdbc.Driver"
                        connectionURL="jdbc:mysql://127.0.0.1/DB_SC_ACTIVITI?useSSL=true" userId="xxxx"
                        password="xxx">
        </jdbcConnection>
        <javaTypeResolver>
            <property name="forceBigDecimals" value="false"/>
        </javaTypeResolver>
        <!--生成Model类存放位置-->
        <javaModelGenerator targetPackage="cn.com.showclear.activiti.pojo.activiti" targetProject="src\main\java">
            <property name="enableSubPackages" value="true"/>
            <property name="trimStrings" value="true"/>
            <!-- 如果是true则会生成构造器的model和mapper -->
            <property name="immutable" value="false"/>
        </javaModelGenerator>
        <!--生成映射文件存放位置-->
        <sqlMapGenerator targetPackage="plan" targetProject="src\main\resources\mappers\activiti">
            <property name="enableSubPackages" value="true"/>
        </sqlMapGenerator>
        <!--生成Dao类存放位置-->
        <javaClientGenerator type="XMLMAPPER" targetPackage="cn.com.showclear.activiti.dao.activiti"
                             targetProject="src\main\java">
            <property name="enableSubPackages" value="true"/>
        </javaClientGenerator>

        <table tableName="T_FORM_TEMPLATE" domainObjectName="FormTemplate" enableCountByExample="false" enableUpdateByExample="false" enableDeleteByExample="false" enableSelectByExample="false" selectByExampleQueryId="false" ></table>

    </context>
</generatorConfiguration>
```
> maven

```
  <!-- mybatis 逆向工程插件 -->
  <plugin>
        <groupId>org.mybatis.generator</groupId>
        <artifactId>mybatis-generator-maven-plugin</artifactId>
        <version>1.3.5</version>
        <configuration>
            <verbose>true</verbose>
            <overwrite>true</overwrite>
        </configuration>
    </plugin>
```

## 接口定义和实现类（供Controller）

> 接口定义

```
/**
 * 表单相关
 *
 * @author YF-XIACHAOYANG
 * @date 2017/12/21 11:38
 */
public interface FormRestServices {
    /**
     * 表单基础接口
     */
    interface BASE {
        RespMapJson selectTest();
    }
}
```

> 服务层实现类

```
/**
 * 表单基础服务
 * @author YF-XIACHAOYANG
 * @date 2018/3/15 16:12
 */
@Service
public class FormBaseServiceImpl implements FormRestServices.BASE {

    @Resource
    private FormTemplateMapper formTemplateMapper;

    @Override
    public RespMapJson selectTest() {
        FormTemplate form = formTemplateMapper.selectByPrimaryKey(Long.valueOf(1));
        System.out.println(form.getContent());
        return new RespMapJson().setData(form);
    }
}
```
FormTemplateMapper  为 mydsatis 逆向工程插件生成

> Controller调用

```
/**
 * 任务查询控制器
 * @author YF-XIACHAOYANG
 * @date 2017/12/20 16:51
 */
@Api(value = "form", description = "工作流表单管理")
@RestController
@RequestMapping("/data/form/")
public class FormRestController {

    @Autowired
    private FormRestServices.BASE formBaseService;

    /**
     * testMybatis
     *
     * @return
     */
    @RequestMapping(value = "/testMybatis", method = RequestMethod.POST)
    public RespMapJson testMybatis()  {
       return formBaseService.selectTest();
    }
}

```

> http 接口测试




![这里写图片描述](https://img-blog.csdn.net/20180326115650117?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzI4NjkwNDE3/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)




至此，可以开始基于mybatis的开发工作了！
    