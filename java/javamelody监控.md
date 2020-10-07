##  引入maven依赖
>```
<dependency>
		<groupId>net.bull.javamelody</groupId>
		<artifactId>javamelody-core</artifactId>
		<version>1.70.0</version>
</dependency>
```

## 配置web.xml
>初始化启动参数增加monitoring-spring.xml
>```
<filter>
        <filter-name>javamelody</filter-name>
        <filter-class>net.bull.javamelody.MonitoringFilter</filter-class>
        <async-supported>true</async-supported>
    </filter>
    <filter-mapping>
        <filter-name>javamelody</filter-name>
        <url-pattern>/monitoring</url-pattern>
        <dispatcher>REQUEST</dispatcher>
        <dispatcher>ASYNC</dispatcher>
    </filter-mapping>
    <filter-mapping>
        <filter-name>javamelody</filter-name>
        <url-pattern>/rest/*</url-pattern>
        <dispatcher>REQUEST</dispatcher>
        <dispatcher>ASYNC</dispatcher>
    </filter-mapping>
    <listener>
        <listener-class>net.bull.javamelody.SessionListener</listener-class>
    </listener>
```

### 监控的地址
>http://ip:port/serverName/monitoring
>
>

###更多的参考
>https://github.com/javamelody/javamelody/wiki


### 问题总结
>1、c3p0数据源怎么监控
>```
 <bean id="mainDataSource" class="net.bull.javamelody.SpringDataSourceFactoryBean">
        <property name="targetName" value="dataSource" />
    </bean>
```