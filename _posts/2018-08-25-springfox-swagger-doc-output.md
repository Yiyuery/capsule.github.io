---
title: Swagger 接口管理和文档导出
date: 2018-08-25 19:22:00
categories:
- Spring Components
tags:
- spring
- springfox-swagger
---

Springfox Swagger 和Spring的整合已经让我们可以动态的生成接口文档了，但是接口文档的生成、管理、导出在网上看了很多博客，着实让我走了很多弯路，都不是很满意。最终总结了一套实际可用的，希望此文不像其他人的文章那般晦涩难懂，给需要的人。

---
# Swagger 接口管理和文档导出

## Swagger 项目接口分组管理、文档生成和批量导出

> 测试用例根据接口分组 批量循环生成对应的 swagger.json

  接口分组管理请前往 《Spring MVC 组件配置 之 RESTFUL API文档以及Mock应用（springfox-swagger）》
  此处分组分为api和ui,api部分为对外提供，ui为前端提供

`SwaggerTest`:

```
package com.xxxx.xx.xxx.web.test;


import org.apache.commons.io.FileUtils;
import org.junit.Before;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.MediaType;
import org.springframework.mock.web.MockHttpServletResponse;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;
import org.springframework.test.context.web.WebAppConfiguration;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.test.web.servlet.MvcResult;
import org.springframework.web.context.WebApplicationContext;

import java.io.BufferedWriter;
import java.nio.charset.StandardCharsets;
import java.nio.file.Files;
import java.nio.file.Paths;

import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;
import static org.springframework.test.web.servlet.setup.MockMvcBuilders.webAppContextSetup;


/**
 * <p>
 *
 * </p>
 *
 * @author xiachaoyang 2018年05月31日 10:21
 * @version V1.0
 * @modificationHistory=========================逻辑或功能性重大变更记录
 * @modify by user: {修改人} 2018年05月31日
 * @modify by reason:{方法名}:{原因}
 */
@RunWith(SpringJUnit4ClassRunner.class)
@WebAppConfiguration
@ContextConfiguration(locations = { "classpath:*-swt.xml"})
public class SwaggerTest {

    @Autowired
    private WebApplicationContext wac;

    private MockMvc mockMvc;

    /**
     * 初始化 MOCK
     */
    @Before
    public void init() {
        mockMvc = webAppContextSetup(wac).build();
    }

    /**
     * 生成 swagger.json
     *
     * @throws Exception
     */
    @Test
    public void getSwaggerJson() throws Exception {
        //获取插件中配置的swagger文件输出地址
        String outputDir = System.getProperty("io.springfox.staticdocs.outputDir");
        //获取插件中配置的swagger.json的访问地址,有几个接口分组就有几个访问地址，地址必须是swagger2controller中原生的，如果是在web.xml自定义的则无法访问，因为mock的服务不会解析web.xml
        String uris = System.getProperty("io.swagger.json.uris");
        //获取插件中配置的每个json文件的名称，名称可配置多个，有几个接口分组就有几个名称， 名称的格式必须是：组件标识-接口分组标识-接口版本号，例如：xxx-api-v1
        String swaggerOutName = System.getProperty("io.swagger.json.output.name");
        String[] uriArray = uris.trim().split(",");
        String[] swaggerOutNameArray = swaggerOutName.trim().split(",");
        //清空文件夹
        //FileUtils.deleteDirectory(Paths.get(outputDir).toFile());
        for (int i = 0; i < uriArray.length; i++) {
            MvcResult mvcResult = mockMvc
                    .perform((get(uriArray[i]))
                            .accept(MediaType.APPLICATION_JSON_UTF8))
                    .andExpect(status().isOk())
                    .andReturn();
            MockHttpServletResponse response = mvcResult.getResponse();
            String swaggerJson = response.getContentAsString();
            Files.createDirectories(Paths.get(outputDir));
            try (BufferedWriter writer = Files.newBufferedWriter(Paths.get(outputDir, swaggerOutNameArray[i]), StandardCharsets.UTF_8)) {
                writer.write(swaggerJson);
            }
        }
    }

}

```

![输入图片说明](https://images.gitee.com/uploads/images/2018/0821/182159_cd66d21f_912956.png "屏幕截图.png")

> 添加配置文件

![输入图片说明](https://images.gitee.com/uploads/images/2018/0821/182140_5c448721_912956.png "屏幕截图.png")

`index.adoc`:

```
include::{generated}/overview.adoc[]
include::{generated}/paths.adoc[]
include::{generated}/security.adoc[]
include::{generated}/definitions.adoc[]
```

`config.properties`:

```
swagger2markup.markupLanguage = ASCIIDOC
swagger2markup.outputLanguage = EN
```

> 生成adoc文件

`Swagger2Markup`:

```java

package com.xxxx.xx.xxx.web.test;

import io.github.swagger2markup.Swagger2MarkupConfig;
import io.github.swagger2markup.Swagger2MarkupConverter;
import io.github.swagger2markup.builder.Swagger2MarkupConfigBuilder;

import java.net.URL;
import java.nio.file.Path;
import java.nio.file.Paths;

import org.apache.commons.configuration2.Configuration;
import org.apache.commons.configuration2.builder.fluent.Configurations;

/**
 * <p>
 *
 * </p>
 *
 * @author xiachaoyang
 * @version V1.0
 * @date 2018年08月21日 9:45
 * @modificationHistory=========================逻辑或功能性重大变更记录
 * @modify By: {修改人} 2018年08月21日
 * @modify reason: {方法名}:{原因}
 * ...
 */
public class Swagger2Markup {

    private static String[] restKeys = new String[]{"ui","api"};//"ui"
    //指定adoc文件生成路径
    private static Path outputDirectory;

    //通过配置文件生成swagger2markup的参数
    public Swagger2MarkupConfig config;

    public Swagger2Markup(String Json) throws Exception{
        //读取配置文件
        Configuration configuration = new Configurations().properties("config.properties");
        config = new Swagger2MarkupConfigBuilder(configuration).build();
        if(Json.startsWith("http")){
            //获取远程json数据
            createAdocFile(new URL(Json));
        }else{
            //获取本地json数据
            createAdocFile(Paths.get(Json));
        }
    }
    /**
     * 通过url生成adoc文件
     */
    public void createAdocFile(URL remoteSwaggerFile){
        Swagger2MarkupConverter.from(remoteSwaggerFile)
                .withConfig(config)
                .build()
                .toFolder(outputDirectory);
    }
    /**
     * 通过json文件生成adoc文件
     */
    public void createAdocFile(Path localSwaggerFile){
        Swagger2MarkupConverter.from(localSwaggerFile)
                .withConfig(config)
                .build()
                .toFolder(outputDirectory);
    }

    public static void main(String[] args) throws Exception{
        //循环生成json对应的acdoc
        for(String key:restKeys){
            outputDirectory = Paths.get("xxx-web/target/asciidoc/generated/"+key);
            //指定本地json文件路径
            new Swagger2Markup("xxx-web/target/swagger/xxx-"+key+"-v1.json");
        }

        //指定远程json文件路径
        //  new Swagger2Markup("http://petstore.swagger.io/v2/swagger.json");

    }
}


```

![输入图片说明](https://images.gitee.com/uploads/images/2018/0821/182811_a26c6b67_912956.png "屏幕截图.png")

> 配置插件执行 生成 pdf 和 html 格式的接口文档

  由于`<phase>compile</phase>`配置，接口分组id不同，调整参数执行mvn compile(或在idea中的maven project中点击可视化命令也可以) 2次即可。

![输入图片说明](https://images.gitee.com/uploads/images/2018/0821/183657_8d8381a5_912956.png "屏幕截图.png")

```
<properties>
        <maven.build.timestamp.format>yyyyMMddHHmmss</maven.build.timestamp.format>
        <rest.api.path>api</rest.api.path>
        <rest.ui.path>ui</rest.ui.path>
        <!--此处切换api和ui-->
        <rest.swagger.path>${rest.ui.path}</rest.swagger.path>
        <asciidoctor.input.directory>${project.basedir}/src/docs/asciidoc</asciidoctor.input.directory>
        <generated.asciidoc.directory>${project.build.directory}/asciidoc/generated</generated.asciidoc.directory>
        <asciidoctor.html.output.directory>${project.build.directory}/asciidoc/html</asciidoctor.html.output.directory>
        <asciidoctor.pdf.output.directory>${project.build.directory}/asciidoc/pdf</asciidoctor.pdf.output.directory>
        <swagger.output.zip>${project.build.directory}/rest-docs/rest-docs_${maven.build.timestamp}</swagger.output.zip>
    </properties>
```

![输入图片说明](https://images.gitee.com/uploads/images/2018/0821/183408_20bbec15_912956.png "屏幕截图.png")

> 文件重命名分类存放

  执行`mvn compile`、`mvn test`分别生成html和pdf的接口文档，文档分类重命名放到指定文件夹（此处对maven生命周期不了解的同学请自行百度）

[maven 插件重命名文件并移动](http://billben.iteye.com/blog/2373011)

`插件配置`：

```
<!--由asciidoc生成pdf：该段插件配置平时注释，只在需要生成文档时解开注释，且前文的操作(swagger.json生成、adoc文件生成)务必已执行完-->
<plugin>
                <groupId>org.asciidoctor</groupId>
                <artifactId>asciidoctor-maven-plugin</artifactId>
                <version>1.5.3</version>
                <dependencies>
                    <dependency>
                        <groupId>org.asciidoctor</groupId>
                        <artifactId>asciidoctorj-pdf</artifactId>
                        <version>1.5.0-alpha.10.1</version>
                    </dependency>
                    <dependency>
                        <groupId>org.jruby</groupId>
                        <artifactId>jruby-complete</artifactId>
                        <version>1.7.21</version>
                    </dependency>
                </dependencies>
                <configuration>
                    <sourceDirectory>${asciidoctor.input.directory}</sourceDirectory>
                    <sourceDocumentName>index.adoc</sourceDocumentName>
                    <attributes>
                        <doctype>book</doctype>
                        <toc>left</toc>
                        <toclevels>3</toclevels>
                        <numbered></numbered>
                        <hardbreaks></hardbreaks>
                        <sectlinks></sectlinks>
                        <sectanchors></sectanchors>
                        <generated>${generated.asciidoc.directory}/${rest.swagger.path}</generated>
                    </attributes>
                </configuration>
                <executions>
                    <execution>
                        <id>output-html</id>
                        <phase>compile</phase>
                        <goals>
                            <goal>process-asciidoc</goal>
                        </goals>
                        <configuration>
                            <backend>html5</backend>
                            <outputDirectory>${asciidoctor.html.output.directory}/${rest.swagger.path}</outputDirectory>
                        </configuration>
                    </execution>
                    <execution>
                        <id>output-pdf</id>
                        <phase>compile</phase>
                        <goals>
                            <goal>process-asciidoc</goal>
                        </goals>
                        <configuration>
                            <backend>pdf</backend>
                            <outputDirectory>${asciidoctor.pdf.output.directory}/${rest.swagger.path}</outputDirectory>
                        </configuration>
                    </execution>
                </executions>
            </plugin>

<!--批量修改文件名 for 接口文档打包-->
            <plugin>
                <groupId>com.coderplus.maven.plugins</groupId>
                <artifactId>copy-rename-maven-plugin</artifactId>
                <executions>
                    <execution>
                        <id>rename-file-pdf-api</id>
                        <phase>test</phase>
                        <goals>
                            <goal>rename</goal>
                        </goals>
                        <configuration>
                            <sourceFile>${asciidoctor.pdf.output.directory}/${rest.api.path}/index.pdf</sourceFile>
                            <destinationFile>${swagger.output.zip}/pdf/xxx-api-${rest.api.version}.pdf</destinationFile>
                        </configuration>
                    </execution>
                    <execution>
                        <id>rename-file-pdf-ui</id>
                        <phase>test</phase>
                        <goals>
                            <goal>rename</goal>
                        </goals>
                        <configuration>
                            <sourceFile>${asciidoctor.pdf.output.directory}/${rest.ui.path}/index.pdf</sourceFile>
                            <destinationFile>${swagger.output.zip}/pdf/xxx-ui-${rest.ui.version}.pdf</destinationFile>
                        </configuration>
                    </execution>
                    <execution>
                        <id>rename-file-html-api</id>
                        <phase>test</phase>
                        <goals>
                            <goal>rename</goal>
                        </goals>
                        <configuration>
                            <sourceFile>${asciidoctor.html.output.directory}/${rest.api.path}/index.html</sourceFile>
                            <destinationFile>${swagger.output.zip}/html/xxx-api-${rest.api.version}.html</destinationFile>
                        </configuration>
                    </execution>
                    <execution>
                        <id>rename-file-html-ui</id>
                        <phase>test</phase>
                        <goals>
                            <goal>rename</goal>
                        </goals>
                        <configuration>
                            <sourceFile>${asciidoctor.html.output.directory}/${rest.ui.path}/index.html</sourceFile>
                            <destinationFile>${swagger.output.zip}/html/xxx-ui-${rest.ui.version}.html</destinationFile>
                        </configuration>
                    </execution>
                    <execution>
                        <id>rename-file-json-api</id>
                        <phase>test</phase>
                        <goals>
                            <goal>rename</goal>
                        </goals>
                        <configuration>
                            <sourceFile>${project.build.directory}/swagger/xxx-api-${rest.api.version}.json</sourceFile>
                            <destinationFile>${swagger.output.zip}/json/xxx-api-${rest.api.version}.json</destinationFile>
                        </configuration>
                    </execution>
                    <execution>
                        <id>rename-file-json-ui</id>
                        <phase>test</phase>
                        <goals>
                            <goal>rename</goal>
                        </goals>
                        <configuration>
                            <sourceFile>${project.build.directory}/swagger/xxx-ui-${rest.ui.version}.json</sourceFile>
                            <destinationFile>${swagger.output.zip}/json/xxx-ui-${rest.ui.version}.json</destinationFile>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
```

![输入图片说明](https://images.gitee.com/uploads/images/2018/0821/184136_9da10b7a_912956.png "屏幕截图.png")

> 然后将文件夹压缩给需要接口的人即可

## REFRENCES

1. [asciidoctor-pdf 【github】](https://github.com/asciidoctor/asciidoctor-pdf/blob/master/docs/theming-guide.adoc)

2. [asciidoctor-maven-plugin 【github】](https://github.com/asciidoctor/asciidoctor-maven-plugin/blob/master/README_zh-CN.adoc)

3. [
通过swagger2markup+asciidoctorj生成html和pdf文档并解决asciidoctorj生成的pdf文件中文显示不全问题（maven方式及java代码方式）](https://blog.csdn.net/qq_25215821/article/details/79175535)
4. [maven打包加时间戳方法总结](https://blog.csdn.net/z410970953/article/details/50680603)

---

## 微信公众号

<center>
<img src="https://images.gitee.com/uploads/images/2018/0717/215030_8e782063_912956.png" width="50%" height="50%"/>
</center>

扫码关注或搜索`架构探险之道`获取最新文章，不积跬步无以至千里，坚持每周一更，坚持技术分享。我和你们一起成长 ^_^ ！
