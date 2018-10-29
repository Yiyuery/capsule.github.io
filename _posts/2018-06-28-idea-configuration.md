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
 * @address:     http://xiazhaoyang.tech
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

## 微信公众号

<center>
<img src="https://images.gitee.com/uploads/images/2018/0717/215030_8e782063_912956.png" width="50%" height="50%"/>
</center>

扫码关注或搜索`架构探险之道`获取最新文章，不积跬步无以至千里，坚持每周一更，坚持技术分享。我和你们一起成长 ^_^ ！
