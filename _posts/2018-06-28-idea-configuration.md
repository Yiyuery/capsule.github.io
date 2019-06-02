---
title: IntelliJ IDEA 配置
date: 2018-06-28 16:22:01
categories:
- IntelliJ IDEA
tags:
- configuration
---


  IntelliJ IDEA 文件模板配置

---


# IntelliJ IDEA 配置

## 文件模板配置

- Class

> 模板

`Project Header(package)`

```
/*
 * @ProjectName: 编程学习
 * @Copyright:   2018 HangZhou Yiyuery Dev, Ltd. All Right Reserved.
 * @address:     https://yiyuery.club
 * @date:        ${DATE} ${TIME}
 * @email:       xiazhaoyang@live.com
 * @description: 本内容仅限于编程技术学习使用，转发请注明出处.
 */
```

`Class File Header(class)`

```
/**
 * <p>
 *
 * </p>
 *
 * @author ${USER}
 * @version V1.0
 * @date ${DATE} ${TIME}
 * @modificationHistory=========================逻辑或功能性重大变更记录
 * @modify By: {修改人} ${DATE}
 * @modify reason: {方法名}:{原因}
 * ...
 */
```

`模板引用`

```
#parse("Project Header.java")
#if (${PACKAGE_NAME} && ${PACKAGE_NAME} != "")package ${PACKAGE_NAME};#end
#parse("File Header.java")
public class ${NAME} {
}
```

![配置示例](https://images.gitee.com/uploads/images/2018/0915/093346_5d2c6468_912956.png "屏幕截图.png")

![输入图片说明](https://images.gitee.com/uploads/images/2018/0915/094327_bb159178_912956.png "屏幕截图.png")

> 配置：

		Setting > Editor > File and Code Templates

		Files : 文件脚本定义
		Includes: 文件引用模板定义

---

## 更多

> 扫码关注“架构探险之道”，回复`文章名称`获取更多源码和文章资源

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190403222309957.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzI4NjkwNDE3,size_16,color_FFFFFF,t_70)

> 知识星球(扫码加入获取源码和文章资源链接)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190403222322267.jpeg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzI4NjkwNDE3,size_16,color_FFFFFF,t_70)
