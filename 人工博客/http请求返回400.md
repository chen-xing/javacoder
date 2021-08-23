### http 400 bad request

### 摘要

一个正常的调用链路，前端经过网关请求后端的一个服务，业务已经正常运行了一两年了，今天突然测试报400了，但是修改请求的url，其他的保持不变，请求又是ok的。究竟是什么原因导致的呢？让我们带着问题进入主题



### 关键字

springboot 400 bad request;常见http状态码介绍



### 1、背景

![问题的调用链路](https://cdn.jsdelivr.net/gh/chen-xing/figure_bed/cdn/20210308202043338.png)

一个正常的调用链路，前端经过网关请求后端的一个服务，业务已经正常运行了一两年了，今天突然测试报400了，但是修改请求的url，其他的保持不变，请求又是ok的。

### 2、初步分析

因为缺乏明确的报错日志，所以只能顺藤摸瓜的方式进行排查了。

+ 既然直接访问是ok的，那么问题大概率是出在网关层的限制。比如网关的安全认证、统一拦截等业务。
+ 路由网关的时候，网关对请求进行了增强，导致真实转发到后端服务的不是原始请求。

### 3、具体的原因

+ 网关对请求进行了增强，本质是增加了一些header，用于会话传递以及调用链跟踪
+ 后端服务对header的总大小有限制，被网关增强后刚好超过限制。

### 4、如何解决

#### 4.1、代码解决

自定义类实现WebServerFactoryCustomizer接口的customize方法

```
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.boot.context.properties.PropertyMapper;
import org.springframework.boot.web.embedded.tomcat.TomcatServletWebServerFactory;
import org.springframework.boot.web.server.WebServerFactoryCustomizer;
import org.springframework.stereotype.Component;

/**
 * 自定义Tomcat容器配置类
 *
 */
@Component
public class MyTomcatWebServerFactoryCustomizer 
        implements WebServerFactoryCustomizer<TomcatServletWebServerFactory> {

    public static final int DEFAULT_MAX_PARAMETER_COUNT = 10000;

    private Logger logger = LoggerFactory.getLogger(getClass());

    /**
     * 单次请求参数最大限制数
     */
    @Value("${server.tomcat.maxParameterCount}")
    private int maxParameterCount = DEFAULT_MAX_PARAMETER_COUNT;

    @Override
    public void customize(TomcatServletWebServerFactory factory) {
        if (logger.isDebugEnabled()) {
            logger.debug("MyTomcatWebServerFactoryCustomizer customize");
        }

        PropertyMapper propertyMapper = PropertyMapper.get();

        propertyMapper.from(this::getMaxParameterCount)
                .when((maxParameterCount) -> maxParameterCount != DEFAULT_MAX_PARAMETER_COUNT)
                .to((maxParameterCount) -> customizerMaxParameterCount(factory, maxParameterCount));
    }

    /**
     * 配置内置Tomcat单次请求参数限制
     *
     * @param factory
     * @param maxParameterCount
     */
    private void customizerMaxParameterCount(TomcatServletWebServerFactory factory, 
                                             int maxParameterCount) {
        factory.addConnectorCustomizers(
                connector -> connector.setMaxParameterCount(maxParameterCount));
    }

    public void setMaxParameterCount(int maxParameterCount) {
        this.maxParameterCount = maxParameterCount;
    }

    public int getMaxParameterCount() {
        return maxParameterCount;
    }
}
```



#### 4.2、配置解决

```
server:
  tomcat:
    max-http-header-size: 16000
    uri-encoding: UTF-8
  max-http-header-size: 16000
两个max-http-header-size的原因是为了兼容不同版本的springboot版本  
```



### 4、其他的4xx、5xx简述

| 状态码 | 可能的原因                                                   |
| ------ | ------------------------------------------------------------ |
| 400    | 1、请求的参数与接口定义不一致，如定义是数组，传入的字符串 2、header的总大小超过限制 |
| 403    | 服务器接收到请求，但是拒绝响应。一般是权限不够               |
| 405    | Menthod Not Allowed,请求的方式不是服务端定义的               |
| 411    | 服务器拒绝接受请求在没有定义Content-Length字段的情况下       |
| 413    | Request Entity Too Large 服务器拒绝处理请求因为请求数据超过服务器能够处理的范围。服务器可能关闭当前连接来阻止客服端继续请求。 |
| 414    | Request-URI Too Long 服务器拒绝服务当前请求因为URI的长度超过了服务器的解析范围。 |
| 415    | 415 Unsupported Media Type 服务器拒绝服务当前请求因为请求数据格式并不被请求的资源支持。 |
| 500    | Internal Server Error 服务器遭遇异常阻止了当前请求的执行     |
| 501    | 501 Not Implemented 服务器没有相应的执行动作来完成当前请求。 |
| 502    | Bad Gateway 作为网关或者代理工作的服务器尝试执行请求时，从上游服务器接收到无效的响应。 |
| 503    | 503 Service Unavailable 因为临时文件超载导致服务器不能处理当前请求。 |
| 504    | 504 Gateway Timeout 作为网关或者代理工作的服务器尝试执行请求时，未能及时从上游服务器（URI标识出的服务器，例如HTTP、FTP、LDAP）或者辅助服务器（例如DNS）收到响应。 |
| 505    | 505 Http Version Not Supported 服务器不支持，或者拒绝支持在请求中使用的 HTTP 版本。这暗示着服务器不能或不愿使用与客户端相同的版本。响应中应当包含一个描述了为何版本不被支持以及服务器支持哪些协议的实体。 |
