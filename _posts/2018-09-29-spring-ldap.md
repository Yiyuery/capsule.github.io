---
title: Spring Boot 整合 LDAP 开发教程
date: 2018-09-29 16:22:01
categories:
- spring
tags:
- ldap
---

  Spring Boot 整合 LDAP 开发教程

---

# Spring Boot 整合 LDAP 开发教程

## 简介

`LDAP`（轻量级目录访问协议，Lightweight Directory Access Protocol)是实现提供被称为目录服务的信息服务。目录服务是一种特殊的数据库系统，其专门针对读取，浏览和搜索操作进行了特定的优化。目录一般用来包含描述性的，基于属性的信息并支持精细复杂的过滤能力。目录一般不支持通用数据库针对大量更新操作操作需要的复杂的事务管理或回卷策略。而目录服务的更新则一般都非常简单。这种目录可以存储包括个人信息、web链结、jpeg图像等各种信息。为了访问存储在目录中的信息，就需要使用运行在TCP/IP 之上的访问协议—LDAP。

LDAP目录中的信息是是按照树型结构组织，具体信息存储在条目(entry)的数据结构中。条目相当于关系数据库中表的记录；条目是具有区别名DN （Distinguished Name）的属性（Attribute），DN是用来引用条目的，DN相当于关系数据库表中的关键字（Primary Key）。属性由类型（Type）和一个或多个值（Values）组成，相当于关系数据库中的字段（Field）由字段名和数据类型组成，只是为了方便检索的需要，LDAP中的Type可以有多个Value，而不是关系数据库中为降低数据的冗余性要求实现的各个域必须是不相关的。LDAP中条目的组织一般按照地理位置和组织关系进行组织，非常的直观。LDAP把数据存放在文件中，为提高效率可以使用基于索引的文件数据库，而不是关系数据库。类型的一个例子就是mail，其值将是一个电子邮件地址。

LDAP的信息是以树型结构存储的，在树根一般定义国家(c=CN)或域名(dc=com)，在其下则往往定义一个或多个组织 (organization)(o=Acme)或组织单元(organizational units) (ou=People)。一个组织单元可能包含诸如所有雇员、大楼内的所有打印机等信息。此外，LDAP支持对条目能够和必须支持哪些属性进行控制，这是有一个特殊的称为对象类别(objectClass)的属性来实现的。该属性的值决定了该条目必须遵循的一些规则，其规定了该条目能够及至少应该包含哪些属性。例如：inetorgPerson对象类需要支持sn(surname)和cn(common name)属性，但也可以包含可选的如邮件，电话号码等属性。

## LDAP 名词解释

o– organization（组织-公司）
ou – organization unit（组织单元-部门）
c - countryName（国家）
dc - domainComponent（域名）
sn – suer name（真实名称）
cn - common name（常用名称


## 配置依赖

```grovvy
compile 'org.springframework.boot:spring-boot-starter-data-ldap'
```

`备注`

  `spring-boot-starter-data-ldap`是`Spring Boot`封装的对LDAP自动化配置的实现，它是基于spring-data-ldap来对LDAP服务端进行具体操作的。


## 连接

- application.yml

```
# LDAP连接配置
spring:
  ldap:
    urls: ldap://10.33.47.7:7003
    base: dc=platform,dc=hikvision,dc=com
    username: ou=acs,ou=componentaccounts,dc=platform,dc=hikvision,dc=com
    password: UlAwRkYl

```

## 查询

- Person.java

  Person中字段为需要从Ldap中查询的数据字段，利用注解@Attribute(name="xx")进行注解,Entry中定义的objectClass和base为Ldap中数据资源的定位信息。查询的时候可以作为返回对象来接收数据。

```java
/*
 * @ProjectName: 编程学习
 * @Copyright:   2018 HangZhou Yiyuery Dev, Ltd. All Right Reserved.
 * @address:     http://xiazhaoyang.tech
 * @date:        2018/7/28 18:15
 * @email:       xiazhaoyang@live.com
 * @description: 本内容仅限于编程技术学习使用，转发请注明出处.
 */
package com.example.chapter3.spring.components.ldap;

import lombok.Data;
import lombok.ToString;
import org.springframework.ldap.odm.annotations.Attribute;
import org.springframework.ldap.odm.annotations.Entry;

import java.util.Date;

/**
 * <p>
 *
 * </p>
 *
 * @author xiachaoyang
 * @version V1.0
 * @date 2018年10月08日 17:16
 * @modificationHistory=========================逻辑或功能性重大变更记录
 * @modify By: {修改人} 2018年10月08日
 * @modify reason: {方法名}:{原因}
 * ...
 */
@Data
@ToString
@Entry(objectClasses = {"bicPersonExt", "bicPerson"}, base = "ou=person,dc=coreservice")
public class Person {
    /**
     * 主键
     */
    @Attribute
    private String personId;

    /**
     * 人员姓名
     */
    @Attribute(name = "cn")
    private String personName;
    /**
     * 组织ID
     */
    @Attribute(name = "orgId")
    private String orgId;
    /**
     * 性别
     */
    @Attribute(name = "sex")
    private Integer sex;
    /**
     * 电话
     */
    @Attribute(name = "mobile")
    private String mobile;
    /**
     * 邮箱
     */
    @Attribute(name = "email")
    private String email;
    /**
     * 工号
     */
    @Attribute(name = "jobNo")
    private String jobNo;
    /**
     * 学号
     */
    @Attribute(name = "studentId")
    private String studentId;

    /**
     * 证件类型
     */
    @Attribute(name = "certType")
    private Integer certType;
    /**
     * 证件号码
     */
    @Attribute(name = "certificateNo")
    private String certNo;

    @Attribute
    protected Date createTime;

    /**
     * 更新时间
     */
    @Attribute
    protected Date updateTime;
    /**
     * 状态
     */
    @Attribute
    protected Integer status;

    @Attribute
    protected Integer disOrder;

    /**
     * 工作单位
     */
    @Attribute
    private String company;
}

```

- IPersonRepo.java

  查询方法的接口层定义(Dao）

```java
package com.example.chapter3.spring.components.ldap;

import org.springframework.ldap.core.LdapTemplate;

import java.util.List;

/**
 * <p>
 *
 * </p>
 *
 * @author xiachaoyang
 * @version V1.0
 * @date 2018年10月08日 15:24
 * @modificationHistory=========================逻辑或功能性重大变更记录
 * @modify By: {修改人} 2018年10月08日
 * @modify reason: {方法名}:{原因}
 * ...
 */
public interface IPersonRepo {

    void setLdapTemplate(LdapTemplate ldapTemplate);

    List<String> getAllPersonNames();

    List<String> getAllPersonNamesWithTraditionalWay();

    List<Person> getAllPersons();

    Person findPersonWithDn(String dn);

    List<String> getPersonNamesByOrgId(String orgId);
}


```

- PersonRepoImpl.java

  Dao接口实现层

```java
/*
 * @ProjectName: 编程学习
 * @Copyright:   2018 HangZhou Yiyuery Dev, Ltd. All Right Reserved.
 * @address:     http://xiazhaoyang.tech
 * @date:        2018/7/28 18:15
 * @email:       xiazhaoyang@live.com
 * @description: 本内容仅限于编程技术学习使用，转发请注明出处.
 */
package com.example.chapter3.spring.components.ldap;

import org.springframework.ldap.core.AttributesMapper;
import org.springframework.ldap.core.LdapTemplate;
import org.springframework.ldap.query.LdapQuery;

import javax.naming.Context;
import javax.naming.NameNotFoundException;
import javax.naming.NamingEnumeration;
import javax.naming.NamingException;
import javax.naming.directory.*;
import java.util.Hashtable;
import java.util.LinkedList;
import java.util.List;

import static org.springframework.ldap.query.LdapQueryBuilder.query;

/**
 * <p>
 *
 * </p>
 *
 * @author xiachaoyang
 * @version V1.0
 * @date 2018年10月08日 15:24
 * @modificationHistory=========================逻辑或功能性重大变更记录
 * @modify By: {修改人} 2018年10月08日
 * @modify reason: {方法名}:{原因}
 * ...
 */
public class PersonRepoImpl implements IPersonRepo {

    private LdapTemplate ldapTemplate;

    @Override
    public void setLdapTemplate(LdapTemplate ldapTemplate) {
        this.ldapTemplate = ldapTemplate;
    }

    /**
     * 查询部分字段集合
     * @return
     */
    @Override
    public List<String> getAllPersonNames() {
        return ldapTemplate.search(
                query().where("objectclass").is("person"), (AttributesMapper<String>) attrs -> (String) attrs.get("cn").get());
    }

    /**
     * 传统LDAP查询方式
     * @return
     */
    @Override
    public List<String> getAllPersonNamesWithTraditionalWay() {
        Hashtable env = new Hashtable();
        env.put(Context.INITIAL_CONTEXT_FACTORY, "com.sun.jndi.ldap.LdapCtxFactory");
        env.put(Context.PROVIDER_URL, "ldap://10.33.47.7:7003/dc=platform,dc=hikvision,dc=com");
        env.put(Context.SECURITY_PRINCIPAL, "ou=acs,ou=componentaccounts,dc=platform,dc=hikvision,dc=com");
        env.put(Context.SECURITY_CREDENTIALS, "UlAwRkYl");
        DirContext ctx;
        try {
            ctx = new InitialDirContext(env);
        } catch (NamingException e) {
            throw new RuntimeException(e);
        }

        List<String> list = new LinkedList<String>();
        NamingEnumeration results = null;
        try {
            SearchControls controls = new SearchControls();
            controls.setSearchScope(SearchControls.SUBTREE_SCOPE);
            results = ctx.search("", "(objectclass=person)", controls);
            while (results.hasMore()) {
                SearchResult searchResult = (SearchResult) results.next();
                Attributes attributes = searchResult.getAttributes();
                Attribute attr = attributes.get("cn");
                String cn = attr.get().toString();
                list.add(cn);
            }
        } catch (NameNotFoundException e) {
            // The base context was not found.
            // Just clean up and exit.
        } catch (NamingException e) {
            //throw new RuntimeException(e);
        } finally {
            if (results != null) {
                try {
                    results.close();
                } catch (Exception e) {
                    // Never mind this.
                }
            }
            if (ctx != null) {
                try {
                    ctx.close();
                } catch (Exception e) {
                    // Never mind this.
                }
            }
        }
        return list;
    }

    /**
     * 查询对象映射集合
     * @return
     */
    @Override
    public List<Person> getAllPersons() {
        return ldapTemplate.search(query()
                .where("objectclass").is("person"), new PersonAttributesMapper());
    }

    /**
     * 根据DN查询指定人员信息
     * @param dn
     * @return
     */
    @Override
    public Person findPersonWithDn(String dn) {
        return ldapTemplate.lookup(dn, new PersonAttributesMapper());
    }

    /**
     * 组装查询语句
     * @param orgId
     * @return
     */
    @Override
    public  List<String> getPersonNamesByOrgId(String orgId) {
        LdapQuery query = query()
                .base("ou=person,dc=coreservice")
                .attributes("cn", "sn")
                .where("objectclass").is("person")
                .and("orgId").is(orgId);
        return ldapTemplate.search(query,(AttributesMapper<String>) attrs -> (String) attrs.get("cn").get());
    }

}


```

- PersonAttributesMapper

  查询辅助对象

```java
/*
 * @ProjectName: 编程学习
 * @Copyright:   2018 HangZhou Yiyuery Dev, Ltd. All Right Reserved.
 * @address:     http://xiazhaoyang.tech
 * @date:        2018/7/28 18:15
 * @email:       xiazhaoyang@live.com
 * @description: 本内容仅限于编程技术学习使用，转发请注明出处.
 */
package com.example.chapter3.spring.components.ldap;

import org.springframework.ldap.core.AttributesMapper;

import javax.naming.NamingException;
import javax.naming.directory.Attributes;

/**
 * <p>
 *
 * </p>
 *
 * @author xiachaoyang
 * @version V1.0
 * @date 2018年10月08日 17:17
 * @modificationHistory=========================逻辑或功能性重大变更记录
 * @modify By: {修改人} 2018年10月08日
 * @modify reason: {方法名}:{原因}
 * ...
 */
public class PersonAttributesMapper implements AttributesMapper<Person> {
    /**
     * Map Attributes to an object. The supplied attributes are the attributes
     * from a single SearchResult.
     *
     * @param attrs attributes from a SearchResult.
     * @return an object built from the attributes.
     * @throws NamingException if any error occurs mapping the attributes
     */
    @Override
    public Person mapFromAttributes(Attributes attrs) throws NamingException {
        Person person = new Person();
        person.setPersonName((String)attrs.get("cn").get());
        person.setOrgId((String)attrs.get("orgId").get());
        return person;
    }
}

```

- LdapTest.java 测试用例

```java
/*
 * @ProjectName: 编程学习
 * @Copyright:   2018 HangZhou Yiyuery Dev, Ltd. All Right Reserved.
 * @address:     http://xiazhaoyang.tech
 * @date:        2018/7/28 18:15
 * @email:       xiazhaoyang@live.com
 * @description: 本内容仅限于编程技术学习使用，转发请注明出处.
 */
package com.example.chapter3.spring.components;


import com.example.chapter3.Chapter3ApplicationTest;
import com.example.chapter3.spring.components.ldap.Person;
import com.example.chapter3.spring.components.ldap.PersonRepoImpl;
import org.junit.Before;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.ldap.core.LdapTemplate;
import org.springframework.test.context.junit4.SpringRunner;

import java.util.List;

/**
 * <p>
 *
 * </p>
 *
 * @author xiachaoyang
 * @version V1.0
 * @date 2018年10月08日 11:47
 * @modificationHistory=========================逻辑或功能性重大变更记录
 * @modify By: {修改人} 2018年10月08日
 * @modify reason: {方法名}:{原因}
 * ...
 */
@RunWith(SpringRunner.class)
@SpringBootTest(classes={Chapter3ApplicationTest.class})
public class LdapTest {

    @Autowired
    private LdapTemplate ldapTemplate;

    private  PersonRepoImpl personRepo;

    @Before
    public void init(){
        personRepo = new PersonRepoImpl();
        personRepo.setLdapTemplate(ldapTemplate);
    }

    @Test
    public void ldapRestTestPart1(){
        // 查询所有人员名称
        //personRepo.getAllPersonNames().forEach(p-> System.out.println(p));
        //荣禧
        //荣耀
        //feng_p1
        //fengzi_0917_1
        //....
        // 查询所有人员集合（指定字段映射）
        //personRepo.getAllPersons().forEach(p-> System.out.println(p.toString()));
        //Person(personId=null, personName=fengzi_0917_7, orgId=14ed2744-fbd4-4868-8ebc-6b0b94d5ae60, sex=null, mobile=null, email=null, jobNo=null, studentId=null, certType=null, certNo=null, createTime=null, updateTime=null, status=null, disOrder=null, company=null)
        //Person(personId=null, personName=fengzi_0917_104, orgId=14ed2744-fbd4-4868-8ebc-6b0b94d5ae60, sex=null, mobile=null, email=null, jobNo=null, studentId=null, certType=null, certNo=null, createTime=null, updateTime=null, status=null, disOrder=null, company=null)

        //根据dn查询
        System.out.println(personRepo.findPersonWithDn("ou=person,dc=coreservice,dc=platform,dc=hikvision,dc=com").toString());

        //根据组织ID查询人员
        //personRepo.getPersonNamesByOrgId("14ed2744-fbd4-4868-8ebc-6b0b94d5ae60").forEach(System.out::println);
        //feng_0925_4687
        //feng_0925_4693
        //...

        //传统查询方式
        //personRepo.getAllPersonNamesWithTraditionalWay().forEach(System.out::println);
        //荣禧
        //荣福
        //feng_p1
        //fengzi_0917_1
        //....

    }
}

```

## 总结

1. LDAP作为轻量级目录查询，利用dn的查询效率十分高
2. LDAP十分适合管理带有树结构的数据
3. LDAP的数据模型（目录设计）十分重要，关乎到数据冗余大小、查询效率等问题。
4. LDAP分页查询需要针对结果循环处理，查询效率一般。


> 实际项目开发中LDAP建议只是作为数据同步中心，因为其对于大数据量的查询效率并不是很高，尤其是数据表变动比较频繁的情况，LDAP中指定大量条件进行模糊搜索时，效率很低。

  通过定时同步任务来对LDAP的数据和本地数据库进行同步，然后本地直接操作数据库表格进行查询、分页，此法对LDAP的依赖性降低。

## REFRENCES

1. [LDAP快速入门](http://www.cnblogs.com/obpm/archive/2010/08/28/1811065.html)
2. [Spring LDAP Refrences Document](https://docs.spring.io/spring-ldap/docs/current/reference/#packaging-overview)

---

## 微信公众号

<center>
<img src="https://images.gitee.com/uploads/images/2018/0717/215030_8e782063_912956.png" width="50%" height="50%"/>
</center>

扫码关注或搜索`架构探险之道`获取最新文章，不积跬步无以至千里，坚持每周一更，坚持技术分享。我和你们一起成长 ^_^ ！
