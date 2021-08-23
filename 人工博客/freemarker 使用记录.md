# freemarker 使用记录

## 1、Long类型显示带逗号

如 id=1,234;

解决的方案是，增加配置 **number_format: '#'**

```
spring:
    profiles:
        active: '@profileActive@'
    application:
        name: blog-web
    freemarker:
        allow-request-override: false
        allow-session-override: false
        cache: false
        charset: UTF-8
        check-template-location: true
        content-type: text/html
        enabled: true
        expose-request-attributes: false
        expose-session-attributes: false
        expose-spring-macro-helpers: true
        prefer-file-system-access: true
        suffix: .ftl
        template-loader-path: classpath:/templates/
        settings:
            template_update_delay: 0
            default_encoding: UTF-8
            classic_compatible: true
            number_format: '#'
```



## 2、boolean类型false返回空

如：var flag=; //当false的时候为空，true的时候为空

```
var isPrivate = ${article.private};//原始的写法
var isPrivate = ${article.private?string("true","false")};//改进的写法

if(isPrivate || isPrivate == 'true') {
      $("#lockModal").modal('show')
}
```

借用网上的一段解释 

> Boolean类型不能使用isXxx，需要使用getXxx ，因为Freemarker使用java会对isXxx映射返回boolean基本型，但是freemarker不支持基本类型boolean，会抛异常。
> freemarker中输出时可以使用这种方式输出${xxx?string("true","flase")}当xxx为true时显示字符串true,否则为字符串false，当然true,false字符串也可以换成其他字符串，比如yes和no。