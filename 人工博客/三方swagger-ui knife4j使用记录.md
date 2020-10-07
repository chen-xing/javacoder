

## 三方swagger-ui knife4j使用记录

### 1、引入pom

```
<dependency>
    <groupId>com.github.xiaoymin</groupId>
    <artifactId>knife4j-spring-ui</artifactId>
    <version>2.0.3</version>
</dependency>
```
**springfox-swagger2这个版本需要注意，否则容易404**

```
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger2</artifactId>
    <version>2.9.2</version>
</dependency>
```

**不需要**引入其他额外的swagger相关的pom,之前因为加多了，导致各种莫名其妙的问题



### 2、编写配置类

```
package tech.chenxing.demo;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Import;
import org.springframework.web.servlet.config.annotation.ResourceHandlerRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurerAdapter;
import springfox.documentation.builders.ApiInfoBuilder;
import springfox.documentation.builders.PathSelectors;
import springfox.documentation.builders.RequestHandlerSelectors;
import springfox.documentation.service.ApiInfo;
import springfox.documentation.service.Contact;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spring.web.plugins.Docket;
import springfox.documentation.swagger2.annotations.EnableSwagger2;

@Configuration
@EnableSwagger2
public class SwaggerConfiguration extends WebMvcConfigurerAdapter {

    private final String SWAGGER_SCAN_BASE_PACKAGE = "com.timevale.sign";

    /**
     * 项目里的静态资源路径指向如下
     *
     * @param registry
     */
    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/doc.html")
                .addResourceLocations("classpath:/META-INF/resources/");
        registry.addResourceHandler("/webjars/**")
                .addResourceLocations("classpath:/META-INF/resources/webjars/");
    }

    @Bean(value = "defaultApi2")
    public Docket defaultApi2() {
        Docket docket =
                new Docket(DocumentationType.SWAGGER_2)
                        .apiInfo(apiInfo())
                        .select()
                        // 这里指定Controller扫描包路径
                        .apis(RequestHandlerSelectors.basePackage(SWAGGER_SCAN_BASE_PACKAGE))
                        .paths(PathSelectors.any())
                        .build();
        return docket;
    }

    private ApiInfo apiInfo() {

        return new ApiInfoBuilder()
                .title("xxx")
                .description("xxx online doc")
                .termsOfServiceUrl("https://www.chenzhuofan.top/")
                .contact(
                        new Contact(
                                "xxxx",
                                "https://www.chenzhuofan.top/",
                                "xxxx"))
                .version("1.0")
                .build();
    }
}

```

需要注意的点

+ package路径需要对
+ 遇到404的时候增加静态文件的注册

然后在**启动类**上ComponentScan增加 配置类的路径



### 3、编写代码

```
入参
@Data
@ApiModel("创建流程文档")
public class AddDocModel extends ToString {
    @ApiModelProperty("文档的名称")
    @NotBlank(message = "name不能为空")
    private String name;

    @ApiModelProperty("文件的fileKey")
    @NotBlank(message = "fileKey不能为空")
    private String fileKey;

    @ApiModelProperty("文件id")
    @NotBlank(message = "fileId不能为空")
    private String fileId;

    @ApiModelProperty("是否加密")
    private Integer encryption;
}

出参
@ApiModel("创建文档的结果集")
@Data
public class AddDocResult extends BaseResult {
    @ApiModelProperty("文档id")
    private String docId;
}

方法
@Api(tags = "文档管理", description = "文档管理")
public class DocServiceImpl implements DocService {
 @ApiOperation(value = "创建文档", httpMethod = "POST")
    public AddDocResult addDoc(@RequestBody AddDocModel addDocModel) {
        //接口实现
    }
}
```



### 4、访问

浏览器 http://localhost:8080/doc.html



### 5、重点问题回顾

+ springfox-swagger2的版本需要高于 2.9.0，这里推荐2.9.2

+ 404 需要重写 WebMvcConfigurerAdapter，增加静态文件注册

+ ```
  org.springframework.context.ApplicationContextException: Failed to start bean 'documentationPluginsBootstrapper'; nested exception is com.google.common.util.concurrent.ExecutionError: java.lang.NoSuchMethodError: com.google.common.collect.FluentIterable.concat(Ljava/lang/Iterable;Ljava/lang/Iterable;)Lcom/google/common/collect/FluentIterable;
  ```

遇到这个报错，原因是项目中的guava版本冲突，需要将guava的版本固定到 20.0，具体的maven冲突问题解决方案参见 [maven冲突解决方案](https://www.94rg.com/article/1773)

### 6、参考链接

+ [swagger-bootstrap-ui使用指南](https://www.94rg.com/article/7)
+ [Knife4j在线doc](https://doc.xiaominfo.com/)