---
title: Swagger 接口管理和文档导出 [Gradle + Spring Boot]
date: 2018-09-27 16:22:01
categories:
- springfox-swagger
tags:
- gradle
---

  Swagger 接口管理和文档导出 [Gradle + Spring Boot]

---

# Swagger 接口管理和文档导出 [Gradle + Spring Boot]

> 基于 Gradle (支持利用`swagger.json`和`gradle`命令行直接生成接口文档并完成打包) [项目内外使用皆可，但是需要是`gradle`进行构建的项目]

## Gradle 配置 [spring boot]

```groovy
buildscript {
    ext {
        springBootVersion = '2.0.5.RELEASE'
    }
    repositories {
        maven { url 'http://maven.aliyun.com/nexus/content/groups/public/' }
        maven { url "https://plugins.gradle.org/m2/" }
        jcenter()
        mavenCentral()
    }
    dependencies {
        classpath("org.springframework.boot:spring-boot-gradle-plugin:${springBootVersion}")
        classpath 'org.asciidoctor:asciidoctor-gradle-plugin:1.5.8'
        classpath 'org.asciidoctor:asciidoctorj-pdf:1.5.0-alpha.10.1'
        classpath 'io.github.swagger2markup:swagger2markup-gradle-plugin:1.2.0'
        classpath "info.robotbrain.gradle.lombok:lombok-gradle:1.1"
    }
}

apply plugin: 'java'
apply plugin: 'idea'
apply plugin: 'org.springframework.boot'
apply plugin: 'io.spring.dependency-management'
apply plugin: 'org.asciidoctor.convert'
apply plugin: 'io.github.swagger2markup'
apply plugin: "info.robotbrain.lombok"
apply plugin: 'groovy'

group = 'com.example'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = 1.8

repositories {
    maven { url 'http://maven.aliyun.com/nexus/content/groups/public/' }
    maven { url "https://plugins.gradle.org/m2/" }
    jcenter()
    mavenCentral()
}


dependencies {
    compileOnly('org.projectlombok:lombok')
    compile('org.springframework.boot:spring-boot-starter')
    compile 'org.codehaus.groovy:groovy-all:2.2.0'
    compile group: 'io.github.swagger2markup', name: 'swagger2markup', version: '1.3.3'
    compile group: 'org.springframework.boot', name: 'spring-boot-configuration-processor', version: '2.0.5.RELEASE'
    testCompile('org.springframework.boot:spring-boot-starter-test')
}

ext {
    asciiDocOutputDir = file("${buildDir}/asciidoc/generated/" + getRestGroupName())
    swaggerOutputDir = file("${buildDir}/swagger")
    springfoxVersion = '2.5.0'
}

/**利用swagger2markup生成asciidoc文件*/
convertSwagger2markup {
    dependsOn test
    swaggerInput "${swaggerOutputDir}/" + getRestGroupName() + ".json"
    outputDir asciiDocOutputDir
    config = [
            'swagger2markup.markupLanguage': 'ASCIIDOC',
            'swagger2markup.outputLanguage': 'EN']
}

/**利用asciidoc文件生成静态文档*/
asciidoctor {
    dependsOn convertSwagger2markup
    sources {
        include 'index.adoc'
    }
    backends = ['html5', 'pdf']
    attributes = [
            doctype    : 'book',
            toc        : 'left',
            toclevels  : '2',
            numbered   : '',
            sectlinks  : '',
            sectanchors: '',
            hardbreaks : '',
            generated  : asciiDocOutputDir
    ]
}

/**清空缓存文件*/
task clearCacheDocs {
    File htmlDir = file("${buildDir}/asciidoc/html5")
    File pdfDir = file("${buildDir}/asciidoc/pdf")
    if (htmlDir.exists()) {
        htmlDir.deleteDir()
    }
    if (pdfDir.exists()) {
        pdfDir.deleteDir()
    }
}


/**复制静态文件到指定目录并修改名称*/
task copyStaticDocsTask() {
    def docHasOut = false
    Thread.start("copyStaticDocsTask") {
        def docPdfDir = new File("${buildDir}/asciidoc/pdf")
        while (!docHasOut) {
            if (null != docPdfDir.listFiles() && docPdfDir.listFiles().length > 0) {
                Thread.currentThread().sleep(3000)
                docHasOut = true
                println "start copy to rest-docs folder..."
                copy {
                    from "${buildDir}/asciidoc/html5"
                    into "${buildDir}/rest-docs/" + getRestGroupName() + "/html5"
                    include "*.html"
                    rename {
                        String fileName ->
                            fileName.replace('index', getRestGroupName())
                    }
                }
                copy {
                    from "${buildDir}/asciidoc/pdf"
                    into "${buildDir}/rest-docs/" + getRestGroupName() + "/pdf"
                    include "*.pdf"
                    rename {
                        String fileName ->
                            fileName.replace('index', getRestGroupName())
                    }
                }
                copy {
                    from "${buildDir}/swagger/"
                    into "${buildDir}/rest-docs/" + getRestGroupName() + "/json"
                    include getRestGroupName() + "*.json"
                }
            }
        }
    }
}

/**获取时间戳*/
def getDate() {
    return new Date().format('yyyyMMddHHmm')
}

/**解析命令行参数：接口分组*/
def getRestGroupName() {
    return project.hasProperty('_rest') ? ext._rest : "xxx-api-v1"
}

/**打包ZIP*/
task archiveReports(type: Zip) {
    from "$buildDir/rest-docs"
    into "rest-docs-zip"
    baseName = "rest-docs-" + getDate()
    destinationDir file("$buildDir/rest-zip")
    doLast {
        delete "$buildDir/rest-docs"
    }
}


/**创建任务起点*/
task createStaticDoc {
      dependsOn asciidoctor
      doFirst {
          println "clearCacheDocs starting..."
          clearCacheDocs
      }
      doLast {
          println getRestGroupName()
          copyStaticDocsTask
      }
}
```

## Gradle Command Line [使用方法]

- `swagger.json`文件放在`build/swagger`目录下
- 使用`gradle`命令指定生成文件

> gradle

```
- gradle createStaticDoc -P_rest="xxx-api-v1"

- gradle createStaticDoc -P_rest="xxx-ui-v1"
/**接口文档打包zip*/
- gradle archiveReports

```

> gradlew

```
- ./gradlew createStaticDoc -P_rest="xxx-api-v1"
- ./gradlew copyStaticDoc -P_rest="xxx-api-v1"

- ./gradlew createStaticDoc -P_rest="xxx-ui-v1"
- ./gradlew archiveReports
```

> Gralde动态传入参数实例

```groovy
def validate() {
    return project.hasProperty('_param') ? ext._param : false
}

task showParam {
    println validate()
}
//命令行运行 gradle showParam -P_param=Abc

```

`运行结果`

```
E:\sync\spring-boot-samples\capsule-utils-web>gradle showParam -P_param=Abc

> Configure project :
Abc

BUILD SUCCESSFUL in 0s

```


## REFERENCES

> Spring Boot

1. [泛型定义](http://www.cnblogs.com/lwbqqyumidi/p/3837629.html)
2. [Java判断对象是否为某一类型的实例](https://blog.csdn.net/ithouse/article/details/52527075)
3. [javaBean与Map<String,Object>互转](https://blog.csdn.net/cuidiwhere/article/details/8130434)
4. [spring boot(5)-properties参数配置](https://blog.csdn.net/ztx114/article/details/78082280)
5. [第三篇: spring-boot中的读取配置文件](https://www.jianshu.com/p/ccde158419cc)
6. [Spring-boot中读取核心配置文件application和自定义properties配置文件的方式](https://blog.csdn.net/qq_23167527/article/details/77977326)
7. [在SpringBoot下读取自定义properties配置文件的方法](https://www.jb51.net/article/130471.htm)
8. [Spring Boot干货系列：（二）配置文件解析](http://www.cnblogs.com/zheting/p/6707036.html)

> Swagger

1. [Github Swagger2Markup/swagger2markup](https://github.com/Swagger2Markup/swagger2markup)
2. [swagger2markup 官方使用手册](http://swagger2markup.github.io/swagger2markup/1.3.3/#extension_commons_content_markup)
3. [asciidoctor/asciidoctor-gradle-plugin](https://github.com/asciidoctor/asciidoctor-gradle-plugin)

> Gradle

1. [彻底理解Gradle的任务](http://www.cnblogs.com/lippi/p/gradle_tasks.html)
2. [创建 Gradle Plugin](http://alighters.com/blog/2016/11/01/create-gradle-plugin/)
3. [Gradle系列教程 W3Cschool](https://www.w3cschool.cn/gradle/3j4r1hu3.html)
4. [Gradle从入门到实战 - Groovy基础](https://blog.csdn.net/singwhatiwanna/article/details/76084580)
5. [Gradle深入与实战（四）自定义集成测试任务](http://benweizhu.github.io/blog/2015/01/31/deep-into-gradle-in-action-4/)
6. [Gradle命令行黑魔法](http://www.cnblogs.com/huang0925/p/3295862.html)
7. [Groovy 线程](https://blog.csdn.net/chennai1101/article/details/55261236)
8. [Gradle 教程说明 用户指南 1~6章](https://blog.csdn.net/jjwwmlp456/article/details/41350413)
9. [Gradle的执行顺序](https://segmentfault.com/q/1010000004503896/a-1020000004712075/revision)
10. [gradle 之 zip](https://blog.csdn.net/cheng545/article/details/79391120)


---

## 更多

> 扫码关注“架构探险之道”，回复`文章名称`获取更多源码和文章资源

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190403222309957.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzI4NjkwNDE3,size_16,color_FFFFFF,t_70)

> 知识星球(扫码加入获取源码和文章资源链接)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190403222322267.jpeg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzI4NjkwNDE3,size_16,color_FFFFFF,t_70)
