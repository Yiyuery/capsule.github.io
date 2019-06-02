---
title: [Spring Security] Spring Security OAuth2（密码模式）
date: 2019-06-01 21:23:00
categories:
- spring
tags:
- spring security
- oauth2
---

本文就OAuth2中客户端授权模式`密码模式`进行深入编码实战。

密码模式（Resource Owner Password Credentials Grant）中，用户向客户端提供自己的用户名和密码。客户端使用这些信息，向"服务商提供商"索要授权。

---

# [Spring Security] Spring Security OAuth2（密码模式）

## 简介

[OAuth](http://en.wikipedia.org/wiki/OAuth)是一个关于授权（authorization）的开放网络标准，在全世界得到广泛应用，目前的版本是2.0版。

本文对OAuth 2.0的设计思路和运行流程，做一个简明通俗的解释，主要参考材料为[RFC 6749](http://www.rfcreader.com/#rfc6749)。

[>参考阮一峰的关于Oauth2的介绍](http://www.ruanyifeng.com/blog/2014/05/oauth_2_0.html)



## 名词定义

- **Third-party application**：第三方应用程序，本文中又称"客户端"（client）。
- **HTTP service**：HTTP服务提供商，本文中简称"服务提供商"。
- **Resource Owner**：资源所有者，本文中又称"用户"（user）。
- **User Agent**：用户代理，本文中就是指浏览器。
- **Authorization server**：认证服务器，即服务提供商专门用来处理认证的服务器。
- **Resource server**：资源服务器，即服务提供商存放用户生成的资源的服务器。它与认证服务器，可以是同一台服务器，也可以是不同的服务器。



## 准备工作

![image-20190602131131370](../../../resources/image/2019-06/02.png)

`spring-security-auth`: 中心认证服务器

`spring-security-resources`: 资源服务器（提供图书相关服务接口）

## OAuth2流程

> 本文就OAuth2中客户端授权模式`密码模式`进行深入编码实战。

密码模式（Resource Owner Password Credentials Grant）中，用户向客户端提供自己的用户名和密码。客户端使用这些信息，向"服务商提供商"索要授权。

在这种模式中，用户必须把自己的密码给客户端，但是客户端不得储存密码。这通常用在用户对客户端高度信任的情况下，比如客户端是操作系统的一部分，或者由一个著名公司出品。而认证服务器只有在其他授权模式无法执行的情况下，才能考虑使用这种模式。

![image-20190602130230999](../../../resources/image/2019-06/01.png)

`它的步骤如下`

（A）用户向客户端提供用户名和密码。

（B）客户端将用户名和密码发给认证服务器，向后者请求令牌。

（C）认证服务器确认无误后，向客户端提供访问令牌。

`B步骤`中，客户端发出的HTTP请求，包含以下参数：

- grant_type：表示授权类型，此处的值固定为"password"，必选项。
- username：表示用户名，必选项。
- password：表示用户的密码，必选项。
- scope：表示权限范围，可选项。

`C步骤中，认证服务器向客户端发送访问令牌`，包含以下参数

- access_token：表示访问令牌，必选项。
- token_type：表示令牌类型，该值大小写不敏感，必选项。
- expires_in：表示过期时间，单位为秒。如果省略该参数，必须其他方式设置过期时间。
- scope：表示权限范围，如果与客户端申请的范围一致，此项可省略。
- state：如果客户端的请求中包含这个参数，认证服务器的回应也必须一模一样包含这个参数。

![image-20190602142143250](../../../resources/image/2019-06/03.png)

其中：

- 获取Token时需要进行`Basic`认证

`http://localhost:8081/authServer/oauth/token?grant_type=password&username=u2&password=12345`

![image-20190602142327790](../../../resources/image/2019-06/04.png)

![image-20190602142649619](../../../resources/image/2019-06/05.png)

比对发现，其实Header中`Authorization`字段中填写的就是`Basic`+`空格`+`Base64(客户端ID:客户端密码)`

- `u2` 和`12345`分别为有权限登录中心认证服务的用户名和密码，用户需要获取资源服务器信息（调用资源获取接口时），会拿着自己的用户名和密码先向中心认证服务获取Token，然后用令牌访问资源服务器的有权限控制的接口。

- 为了验证资源服务器有对自己的资源做保护，我们先发起一个获取图书信息的请求。

![image-20190602143342548](../../../resources/image/2019-06/image-20190602143342548.png)

- 然后我们用获取到的Token再尝试发起一次请求

```json
{
    "access_token": "61ae35ff-d47d-46d8-ba27-2b7fabd30c50",
    "token_type": "bearer",
    "refresh_token": "358b62f2-be22-48d5-8bd7-31ff3da1f8cb",
    "expires_in": 1199,
    "scope": "book_info"
}
```

![image-20190602143645001](../../../resources/image/2019-06/image-20190602143645001.png)

其中请求头中需保证`Authorization`的值为`Bearer`+`空格`+`access_token的值`

- Token过期后请求

![image-20190602153255218](../../../resources/image/2019-06/image-20190602153255218.png)

 ## 核心配置类

> 中心认证服务器关键配置

```java
/**
     * ClientDetailsServiceConfigurer 能够使用内存或 JDBC 方式实现获取已注册的客户端详情，有几个重要的属性：
     * clientId：客户端标识 ID
     * secret：客户端安全码
     * scope：客户端访问范围，默认为空则拥有全部范围
     * authorizedGrantTypes：客户端使用的授权类型，默认为空
     * authorities：客户端可使用的权限
     *
     * @param clients
     * @throws Exception
     */
    @Override
    public void configure(final ClientDetailsServiceConfigurer clients) throws Exception {

        clients.inMemory()
                .withClient("SampleClientId")
                .secret(passwordEncoder.encode("secret"))
                .authorizedGrantTypes("authorization_code")
                .scopes("user_info")
                .autoApprove(true)
                .redirectUris("http://localhost:8082/client/login", "http://localhost:8083/client2/login", "http://www.example.com/")

                .and()
                .withClient("BookResourceClientId")
                .secret(passwordEncoder.encode("secret"))
                .authorizedGrantTypes("password","refresh_token")
                .scopes("book_info")
                .resourceIds("book_rest_api")
                .accessTokenValiditySeconds(1200)
                .refreshTokenValiditySeconds(50000);
    }
```

`关键类`

 **BasicAuthenticationFilter**会获取header中的Authorization Basic，提取出客户端信息。

```java
@Override
	protected void doFilterInternal(HttpServletRequest request,
			HttpServletResponse response, FilterChain chain)
					throws IOException, ServletException {
		final boolean debug = this.logger.isDebugEnabled();

		String header = request.getHeader("Authorization");

		if (header == null || !header.toLowerCase().startsWith("basic ")) {
			chain.doFilter(request, response);
			return;
		}

		try {
			String[] tokens = extractAndDecodeHeader(header, request);
			assert tokens.length == 2;

			String username = tokens[0];

			if (debug) {
				this.logger
						.debug("Basic Authentication Authorization header found for user '"
								+ username + "'");
			}

			if (authenticationIsRequired(username)) {
				UsernamePasswordAuthenticationToken authRequest = new UsernamePasswordAuthenticationToken(
						username, tokens[1]);
				authRequest.setDetails(
						this.authenticationDetailsSource.buildDetails(request));
				Authentication authResult = this.authenticationManager
						.authenticate(authRequest);

				if (debug) {
					this.logger.debug("Authentication success: " + authResult);
				}

				SecurityContextHolder.getContext().setAuthentication(authResult);

				this.rememberMeServices.loginSuccess(request, response, authResult);

				onSuccessfulAuthentication(request, response, authResult);
			}

		}
		catch (AuthenticationException failed) {
			SecurityContextHolder.clearContext();

			if (debug) {
				this.logger.debug("Authentication request for failed: " + failed);
			}

			this.rememberMeServices.loginFail(request, response);

			onUnsuccessfulAuthentication(request, response, failed);

			if (this.ignoreFailure) {
				chain.doFilter(request, response);
			}
			else {
				this.authenticationEntryPoint.commence(request, response, failed);
			}

			return;
		}

		chain.doFilter(request, response);
	}
```

访问`/oauth/token`，先验证了client信息，并作为`authentication`存储在`SecurityContextHolder`中。传递到`TokenEndPoint`的`principal`是client，paramters包含了user的信息和grantType。

![image-20190602153728310](../../../../resources/image/2019-06/image-20190602153728310.png)

> 资源服务器关键配置

**ResourceSecurityConfig**

```java
/*
 * @ProjectName: 编程学习
 * @Copyright:   2019 HangZhou xiazhaoyang Dev, Ltd. All Right Reserved.
 * @address:     http://xiazhaoyang.tech
 * @date:        2019/5/20 20:57
 * @email:       xiazhaoyang@live.com
 * @description: 本内容仅限于编程技术学习使用，转发请注明出处.
 */
package com.example.repo;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.security.oauth2.config.annotation.web.configuration.EnableResourceServer;
import org.springframework.security.oauth2.config.annotation.web.configuration.ResourceServerConfigurerAdapter;
import org.springframework.security.oauth2.config.annotation.web.configurers.ResourceServerSecurityConfigurer;
import org.springframework.security.oauth2.provider.token.RemoteTokenServices;
import org.springframework.security.oauth2.provider.token.ResourceServerTokenServices;
import org.springframework.security.web.AuthenticationEntryPoint;

import javax.annotation.Resource;

/**
 * <p>
 * 资源服务配置
 * ResourceServerConfigurerAdapter用于保护oauth要开放的资源，同时主要作用于client端以及token的认证(Bearer auth)
 * </p>
 *
 * @author xiazhaoyang
 * @version v1.0.0
 * @date 2019/5/20 20:57
 * @modificationHistory=========================逻辑或功能性重大变更记录
 * @modify By: {修改人} 2019/5/20
 * @modify reason: {方法名}:{原因}
 * ...
 */
@EnableResourceServer
@Configuration
public class ResourceSecurityConfig extends ResourceServerConfigurerAdapter {

    @Value("${security.oauth2.resource.id}")
    public String resourceId;

    @Resource
    public RemoteTokenServices remoteTokenServices;

    /**
     * 资源服务器承载资源[REST API]，客户端感兴趣的资源位于  /book/ 。
     *
     * @param http
     * @throws Exception
     * @EnableResourceServer注释，适用在OAuth2资源服务器， 实现了Spring Security的过滤器验证的请求传入OAuth2令牌。 
     * ResourceServerConfigurerAdapter类实现 ResourceServerConfigurer 提供的方法来
     * 调整 OAuth2安全保护的访问规则和路径。
     */
    @Override
    public void configure(HttpSecurity http) throws Exception {
        http.csrf().disable();
        http.requestMatchers().antMatchers("/book/**")
                .and()
                .authorizeRequests()
                .antMatchers("/book/**").authenticated();
    }

    @Override
    public void configure(ResourceServerSecurityConfigurer resources) throws Exception {
        //如果关闭 stateless，则 accessToken 使用时的 session id 会被记录，后续请求不携带 accessToken 也可以正常响应
        resources.resourceId(resourceId).stateless(true).tokenServices(remoteTokenServices);
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```

**BookResourceController**

```java
/*
 * @ProjectName: 编程学习
 * @Copyright:   2019 HangZhou xiazhaoyang Dev, Ltd. All Right Reserved.
 * @address:     http://xiazhaoyang.tech
 * @date:        2019/5/26 21:50
 * @email:       xiazhaoyang@live.com
 * @description: 本内容仅限于编程技术学习使用，转发请注明出处.
 */
package com.example.repo;

import org.springframework.security.access.prepost.PreAuthorize;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

/**
 * <p>
 *
 * </p>
 *
 * @author xiazhaoyang
 * @version v1.0.0
 * @date 2019/5/26 21:50
 * @modificationHistory=========================逻辑或功能性重大变更记录
 * @modify By: {修改人} 2019/5/26
 * @modify reason: {方法名}:{原因}
 * ...
 */
@RestController
public class BookResourceController {

    @GetMapping("/book/info")
    @PreAuthorize("hasAnyAuthority('USER')")
    public String getBookInfoById(String id) {
        return String.format("get book info by id:%s", id);
    }
}
```

**application.yaml**

```yaml
server:
  port: 8084
  servlet:
    context-path: /book-resources
    session:
      cookie:
        name: BOOKRESOURCE

management:
  security:
    enabled: false

# 是否开启基本的鉴权，默认为true。 true：所有的接口默认都需要被验证，将导致 拦截器[对于 excludePathPatterns()方法失效]
security:
  basic:
    enabled: false

  oauth2:
    client:
      client-id: BookResourceClientId
      client-secret: secret
      access-token-uri: http://localhost:8081/authServer/oauth/token
      user-authorization-uri: http://localhost:8081/authServer/oauth/authorize
      user-logout-uri: http://localhost:8081/authServer/logout
    resource:
      id: book_rest_api
      preferTokenInfo: true
      token-info-uri: http://localhost:8081/authServer/oauth/check_token
      filter-order: 3

endpoints:
  health:
    sensitive: false
    enabled: true

spring:
  thymeleaf:
    cache: false

```

`关键类`

**OAuth2AuthenticationProcessingFilter**

```java
public void doFilter(ServletRequest req, ServletResponse res, FilterChain chain) throws IOException,
			ServletException {

		final boolean debug = logger.isDebugEnabled();
		final HttpServletRequest request = (HttpServletRequest) req;
		final HttpServletResponse response = (HttpServletResponse) res;

		try {
			//从request中提取Token
			Authentication authentication = tokenExtractor.extract(request);

			if (authentication == null) {
				if (stateless && isAuthenticated()) {
					if (debug) {
						logger.debug("Clearing security context.");
					}
					SecurityContextHolder.clearContext();
				}
				if (debug) {
					logger.debug("No token in request, will continue chain.");
				}
			}
			else {
				request.setAttribute(OAuth2AuthenticationDetails.ACCESS_TOKEN_VALUE, authentication.getPrincipal());
				if (authentication instanceof AbstractAuthenticationToken) {
					AbstractAuthenticationToken needsDetails = (AbstractAuthenticationToken) authentication;
					needsDetails.setDetails(authenticationDetailsSource.buildDetails(request));
				}
                // OAuth2AuthenticationManager 验证PreAuthenticatedAuthenticationToken
				Authentication authResult = authenticationManager.authenticate(authentication);

				if (debug) {
					logger.debug("Authentication success: " + authResult);
				}

				eventPublisher.publishAuthenticationSuccess(authResult);
				SecurityContextHolder.getContext().setAuthentication(authResult);

			}
		}
		catch (OAuth2Exception failed) {
			SecurityContextHolder.clearContext();

			if (debug) {
				logger.debug("Authentication request failed: " + failed);
			}
			eventPublisher.publishAuthenticationFailure(new BadCredentialsException(failed.getMessage(), failed),
					new PreAuthenticatedAuthenticationToken("access-token", "N/A"));

			authenticationEntryPoint.commence(request, response,
					new InsufficientAuthenticationException(failed.getMessage(), failed));

			return;
		}

		chain.doFilter(request, response);
	}
```

**BearerTokenExtractor**

![image-20190602182920409](../../../resources/image/2019-06/image-20190602182920409.png)

```java
public PreAuthenticatedAuthenticationToken(Object aPrincipal, Object aCredentials) {
		super(null);
		this.principal = aPrincipal;
		this.credentials = aCredentials;
	}
```

`BearerTokenExtractor`解析request，`extractToken`方法从header参数`Authorization` **Bearer  [tokenValue]**中抽取token，并返回一个`principal`值为token的`PreAuthenticatedAuthenticationToken`对象。

再根据**OAuth2AuthenticationManager**校验`authentication`的合法性。

> 对于本例中，资源服务器和中心认证服务是分离开的，所以还需进行Token的校验

![image-20190602183954397](../../../resources/image/2019-06/image-20190602183954397.png)

当请求资源服务器的时候，在通过`OAuth2AuthenticationManager`校验完后`authentication`合法性后，还会调用中心认证服务的`/oauth/check_token`接口进行token的校验。

Tips: 这些都依赖于资源服务器的yaml文件中配置的路由

```yaml
security:
  basic:
    enabled: false

  oauth2:
    client:
      client-id: BookResourceClientId
      client-secret: secret
      access-token-uri: http://localhost:8081/authServer/oauth/token
      user-authorization-uri: http://localhost:8081/authServer/oauth/authorize
      user-logout-uri: http://localhost:8081/authServer/logout
    resource:
      id: book_rest_api
      preferTokenInfo: true
      token-info-uri: http://localhost:8081/authServer/oauth/check_token
      filter-order: 3
```

## 总结

本文总结了基于Spring Security 和 OAuth2的密码授权模式的主要流程和关键节点的参数。并提供了将资源服务器和中心认证服务器分开的配置方案。这样做的主要目的是考虑到现实场景中，往往各个模块的职责是单一的，当资源模型较多时，分离部署明显是更好的方式。


## REFRENCES

- [OAuth2 源码分析(三.密码模式源码）](https://blog.csdn.net/qq_30905661/article/details/82704746)

- [OAuth2整合redis和mysql](https://github.com/zth390872451/oauth2-redis-mysql)
- [Spring Boot 与 OAuth2](http://www.spring4all.com/article/827)
- [Spring 官网OAuth2开发指南](https://projects.spring.io/spring-security-oauth/docs/oauth2.html)
