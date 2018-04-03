---
title: Spring Boot 整合 WebSocket 开发笔记
date: 2018-04-04 00:28:00
categories:
- WebSocket
tags:
- websocket
- spring-boot
---

WebSocket为浏览器和服务器之间提供了双工异步通信功能，也就是说我们可以利用浏览器给服务器发送消息，服务器也可以给浏览器发送消息，目前主流浏览器的主流版本对WebSocket的支持都算是比较好的，但是在实际开发中使用WebSocket工作量会略大，而且增加了浏览器的兼容问题，这种时候我们更多的是使用WebSocket的一个子协议stomp，利用它来快速实现我们的功能。OK，关于WebSocket我这里就不再多说，我们主要看如何使用，如果小伙伴们有兴趣可以查看这个回答来了解更多关于WebSocket的信息[WebSocket 是什么原理？为什么可以实现持久连接](https://www.zhihu.com/question/20215561)

---

# Spring Boot 整合 WebSocket 开发笔记

## Maven pom

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-websocket</artifactId>
</dependency>
```

## Configuration

```
package cn.com.showclear.config;

import org.springframework.context.annotation.Configuration;
import org.springframework.messaging.simp.config.MessageBrokerRegistry;
import org.springframework.web.socket.config.annotation.AbstractWebSocketMessageBrokerConfigurer;
import org.springframework.web.socket.config.annotation.EnableWebSocketMessageBroker;
import org.springframework.web.socket.config.annotation.StompEndpointRegistry;

/**
 * Websocket 整合Springboot 开发
 * <p>
 * Target:
 * > 1、预案删除后台推送预案运行界面自动刷新
 *
 * @author YF-XIACHAOYANG
 * @date 2017/11/23 13:49
 */
@Configuration
//EnableWebSocketMessageBroker注解表示开启使用STOMP协议来传输基于代理的消息，Broker就是代理的意思。
@EnableWebSocketMessageBroker
public class WebSocketConfig extends AbstractWebSocketMessageBrokerConfigurer {

    /**
     * 注册STOMP协议节点，同时指定使用SockJS协议
     * @param registry
     */
    @Override
    public void registerStompEndpoints(StompEndpointRegistry registry) {
        registry.addEndpoint("/endpointSang").withSockJS();
    }

    /**
     * 配置消息代理，由于我们是实现推送功能，这里的消息代理是/msg/...
     * @param registry
     */
    @Override
    public void configureMessageBroker(MessageBrokerRegistry registry) {
        registry.enableSimpleBroker("/msg/");
    }
}

```

## Controller

> 消息体

    1、RequestMessage
```
package cn.com.showclear.common.utils;

/**
 * 浏览器发送消息的接收类
 * 浏览器发送来的消息用这个类来接收
 * @author YF-XIACHAOYANG
 * @date 2017/11/23 14:30
 */
public class RequestMessage {
    private String name;

    public String getName() {
        return name;
    }
}

```

    2、ResponseMessage
```
package cn.com.showclear.common.utils;

/**
 * 响应消息类
 * 服务器返回给浏览器的消息由这个类来承载
 * @author YF-XIACHAOYANG
 * @date 2017/11/23 14:30
 */
public class ResponseMessage {
    private String responseMessage;

    public ResponseMessage(String responseMessage) {
        this.responseMessage = responseMessage;
    }

    public String getResponseMessage() {
        return responseMessage;
    }
}

```

> 请求控制器

```
package cn.com.showclear.common.controller;

import cn.com.showclear.common.utils.RequestMessage;
import cn.com.showclear.common.utils.ResponseMessage;
import org.springframework.messaging.handler.annotation.MessageMapping;
import org.springframework.messaging.handler.annotation.SendTo;
import org.springframework.stereotype.Controller;

/**
 * Websocket 请求控制器
 * 负责后台接收推送过来的消息并完成推送的交互
 * @author YF-XIACHAOYANG
 * @date 2017/11/23 14:26
 */
@Controller
public class WsController {

    //接收主题
    @MessageMapping("/hello")
    //推送主题
    @SendTo("/msg/resp")
    public ResponseMessage say(RequestMessage message) {
        System.out.println(message.getName());
        return new ResponseMessage("hello," + message.getName() + " !");
    }
}

```

## 前端脚本
    
    1、STOMP协议的客户端脚本stomp.js、
    2、SockJS的客户端脚本sock.js
    3、jQuery

> 建立连接和主题推送
 
```     

/*连接、订阅主题回调*/
 function connect() {
        var socket = new SockJS('/endpointSang');
        stompClient = Stomp.over(socket);
        stompClient.connect({}, function (frame) {
            setConnected(true);
            console.log('Connected:' + frame);
            stompClient.subscribe('/msg/resp', function (response) {
                showResponse(JSON.parse(response.body).responseMessage);
            })
        });
    }

/*推送*/
  function sendName() {
        var name = $('#name').val();
        console.log('name:' + name);
        stompClient.send("/hello", {}, JSON.stringify({'name': name}));
    }
    
```

## 推送框架设计

> 不使用@SendTo注解，通过SimpMessagingTemplate完成消息推送服务层的搭建，
   
    1、@SendTo 适合放在WebsocketListener[@Controller]中监听指定指定消息代理并完成任务转发
    2、notifyService 适合在不同控制层[@Controller]中直接完成推送服务


## 后端推送服务搭建

> service

```

/**
 * 消息推送服务
 */
interface NOTIFY{
    /**
     * 通知消息
     * @param noticeVO
     */
    void notice(NoticeVO noticeVO);
}

```

> impl

```
package cn.com.showclear.plan.impl.common;

import cn.com.showclear.plan.pojo.common.NoticeVO;
import cn.com.showclear.plan.service.common.BaseServices;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.messaging.simp.SimpMessagingTemplate;
import org.springframework.stereotype.Service;

/**
 * 消息推送服务
 * @author YF-XIACHAOYANG
 * @date 2017/11/23 15:36
 */
@Service
public class NotificationServiceImpl implements BaseServices.NOTIFY {

    private static final Logger log = LoggerFactory.getLogger(NotificationServiceImpl.class);

    @Autowired
    private SimpMessagingTemplate messagingTemplate;

    /**
     * 通知消息发送
     *
     * @param noticeVO
     */
    @Override
    public void notice(NoticeVO noticeVO) {
        try {
            messagingTemplate.convertAndSend(noticeVO.getSubject(), noticeVO.getData());
        } catch (Exception e) {
            log.error("notice msg error.", e);
        }
    }
}
```

> NoticeVO 消息体基本对象

```
package cn.com.showclear.plan.pojo.common;

/**
 * 消息通知对象
 *
 * @author LuLihong
 * @date 2017-08-30
 **/
public class NoticeVO {
    private final String subject;
    private final Object data;

    public NoticeVO(String subject, Object data) {
        this.subject = subject;
        this.data = data;
    }

    public Object getData() {
        return data;
    }

    public String getSubject() {
        return subject;
    }
}
```

> eg
    
```

......

@Autowired
private BaseServices.NOTIFY notifyService;

......

/**
 * 预案结束
 *
 * @return
 */
@RequestMapping(value = "/finishPlan", method = RequestMethod.POST)
public RespMapJson finishPlan(Integer planReId) {
    DataSourceTypeManager.set(DataSources.PLAN);
    RespMapJson resp = psmRunService.finishPlan(planReId);
    if (resp.getCode() == 0) {
        //通知预案在运行界面刷新
        resp.put("isRefresh", true);
        notifyService.notice(new NoticeVO(NoticeSubject.MSG_REFRESH, resp.getResp()));
    }
    return resp;
}
```
    
## 前端推送管理和回调监听


> amd & require

```
/**
 * require 通用配置
 * @author Yiyuery
 */
require.config({
    baseUrl: window.main.contextPath,
    paths: {
      
        jquery: "js/lib/jquery/jquery-1.9.1",

        //websocket
        stomp: "js/websocket/stomp",
        sockjs: "js/websocket/sockjs.min",

        //ws-utils
        'scooper-notice': 'js/scooper/scooper.notice',
        'msg-ws':'js/websocket/msg-websocket',
        
        //提示组件
        layer: 'js/lib/layer/layer',
        
        //自定义组件
        capsule: 'js/lib/capsule/capsule.util',
    },
    /*定义模块依赖*/
    shim: {
        layer: { deps: ['jquery'] },
        capsule: { deps: ['jquery', 'layer', 'pager'] }
    }
});
```

> 推送管理

```
/**
 * Created by LuLihong on 2017/8/30.
 */
window.scooper = window.scooper || {};
/**
 * 消息通知
 * @type {{}}
 */
window.scooper.notice = {
    /**
     * 主题
     */
    subjects: {
        /**预案运行数目变更通知主题*/
        refresh: '/msg/refresh',
        /**测试监听主题**/
        test:'/msg/test',
        /**测试通道占用**/
        topic:'/topic/resp'
    },

    /**
     * 主题监听器
     */
    listeners: {},

    /**
     * 获取需要监听的主题
     * @returns {[*,*,*]}
     */
    getSubjects: function() {
        return [this.subjects.refresh,this.subjects.test,this.subjects.topic];
    },

    /**
     * 添加监听器
     * @param subject
     * @param listener
     */
    addListener: function(subject, listener) {
        this.listeners[subject] = listener;
    },

    /**
     * 删除监听器
     * @param subject
     */
    removeListener: function(subject) {
        delete this.listeners[subject];
    },

    /**
     * 获取监听器
     * @param subject
     * @returns {*}
     */
    getListener: function(subject) {
        return this.listeners[subject];
    },

    /**
     * 接收到通知
     * @param subject
     * @param notice
     */
    recvNotice: function(subject, notice) {
        window.console.log('recv notice: ' + subject + '=' + notice);
        var listener = this.listeners[subject];
        if (listener) {
            var noticeObj = JSON.parse(notice);
            listener(noticeObj);

        }
    }
};
```

> 推送核心模块

```
/**
 * 获取后台的通知消息，以websocket方式获取。
 * Created by LuLihong on 2017/8/30.
 */
define(['jquery', 'scooper-notice', 'stomp', 'sockjs'], function ($) {

    var stompClient = null;

    /**
     * 定义模块
     * @type {{}}
     */
    var MSGWS = {
        /**
         * 创建SocKJS实例并获取websocket连接
         */
        connect: function (fn) {
            //申明连接的SockJS的endpoint名称：与后台WebsocketConfig保持一致
            var socket = new SockJS(window.main.contextPath + '/endpointSang', null, {rtt: 5000});
            //使用STOMP来创建WebSocket客户端
            stompClient = Stomp.over(socket);
            stompClient.connect({}, function (frame) {
                var subjects = window.scooper.notice.getSubjects();
                $.each(subjects, function (i, subject) {
                    /**
                     * 订阅/msg/resp等主题发送来的消息，分发事件
                     * Controller中的say方法上添加的@SendTo注解的参数
                     * stompClient中的send方法表示发送一条消息到服务端
                     */
                    stompClient.subscribe(subject, function (resp) {
                        window.scooper.notice.recvNotice(subject, resp.body);
                    });
                });
            });
            if (fn instanceof Function) {
                fn();
            }
        },
        /**
         * 断开连接
         */
        disconnect: function () {
            if (stompClient != null) {
                stompClient.disconnect();
            }

            window.console.log('Disconnected');
        },
        /**
         * 检查连接状态
         */
        checkState: function () {
            if (stompClient == null || !stompClient.connected) {
                MSGWS.connect();
            }
        },
        /**
         * 连接保持定时监听
         */
        keepListenerTimer: function () {
            setInterval(MSGWS.checkState, 5000);
        },
        /**
         * 初始化
         */
        init: function () {
            MSGWS.connect(function () {
                MSGWS.keepListenerTimer();
            });

        }
    };

    /*立即执行函数，完成连接*/
    MSGWS.init();

    /**
     * 对外开放部分接口
     */
    return {
        /**
         * 关闭连接
         */
        disconnect: MSGWS.disconnect
    }
});
```

> 注册回调监听

```
/*头部引入模块*/
define(["require", "exports", "jquery", "avalon", "capsule", "layer", 'msg-ws'], function (require, exports, $, avalon, capsule, layer, msgWs)


/**
 * 注册websocket回调主题
 */
function regWebSocketListener() {
    window.scooper.notice.addListener('/msg/refresh', wsNotify);
},
    
/**
 * websocket消息通知
 * @param msg
 */
function wsNotify(isRefresh) {
    if (isRefresh) {
        layer.msg('执行中预案数目发生变更，正在重新加载...');
        capsule.baseUtil.delay(2, function () {
            window.location.reload();
        });
    }
}

```
    



