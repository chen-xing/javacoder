



## springboot集成swagger

### 1、为什么需要swagger

> + 开发人员频繁的修改服务端的rest接口，而对接人员和测试人员未能在第一时间获取到最新的文档
> + 接口编写完成，需要再花一定的时间去按照模板编写接口文档费事费力，不如编写代码来的轻松

+ 如果你也有同感，那么swagger绝对是你的救星



### 2、swagger是什么

> + swagger是一个基于注解生成在线api文档的工具包。有了它编写好代码，接口的入参、出参、方法定义上打上指定的注解，辅以描述，那么框架会根据规范生成格式规范的api在线文档，与代码自动保持同步，所见即所得。
> + 一个在线的postman,大部分参数为你格式化好了，你只需把形参替换成实参就可以发送请求了
> + 支持导出markdown、pdf、word等第三方格式的文档
> + 自动标识最新的api,改动一目了然



### 3、如何集成

#### a、引入pom

```
 <!--swagger2-->
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger2</artifactId>
            <version>2.8.0</version>
        </dependency>

        <dependency>
            <groupId>com.github.xiaoymin</groupId>
            <artifactId>swagger-bootstrap-ui</artifactId>
            <version>1.8.7</version>
        </dependency>
```



+ 需要说明的是这里采用的是第三方的swagger-ui，相较于官网的版本，界面更优美，体验更好。ui只是一个插件，可以无缝替换

#### b、代码上增加注解

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

+ 重要注解说明

  > ApiModel 用来定义接口的出参和入参的实体类
  >
  > ApiModelProperty 定义实体类的属性值
  >
  > Api 用来定义api功能模块
  >
  > ApiOperation 定义一个具体的api
  >
  >  httpMethod = "POST" 接口只生成POST请求的文档
  >
  > @RequestBody 入参采用json

#### c、配置swagger

```
@Configuration
@EnableSwagger2
public class SwaggerConfiguration extends WebMvcConfigurerAdapter {
    private final String SWAGGER_SCAN_BASE_PACKAGE = "top.zhuofan.datafly";

    /**
     * 项目里的静态资源路径指向如下
     *
     * @param registry
     */
    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("swagger-ui.html")
                .addResourceLocations("classpath:/META-INF/resources/");
        registry.addResourceHandler("/webjars/**")
                .addResourceLocations("classpath:/META-INF/resources/webjars/");
        super.addResourceHandlers(registry);
    }

    @Override
    public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
        configurer.enable();
    }


    @Bean
    public Docket createRestApi() {

        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .select()
                .apis(RequestHandlerSelectors.basePackage(SWAGGER_SCAN_BASE_PACKAGE))
                .paths(PathSelectors.any())
                .build();
    }

    private ApiInfo apiInfo() {

        return new ApiInfoBuilder()
                .title("卓帆网")
                .description("datafly online doc")
                .termsOfServiceUrl("https://www.chenzhuofan.top/")
                .contact(new Contact("", "", "308679291@qq.com"))
                .version("1.0")
                .build();
    }
}

```

######  

+ 配置的主要内容是

  > + 将swagger-ui的静态页相关的资源添加到springboot可访问的路径下
  > + 添加swagger的扫描包路径
  > + 添加swagger的版权、开发人员的联系信息

### 4、效果展示

![](https://img2018.cnblogs.com/blog/493641/201904/493641-20190426214421258-996930423.png)



![](https://img2018.cnblogs.com/blog/493641/201904/493641-20190426214433784-1089309906.png)

### 5、后续

更多精彩，敬请关注， [ 程序员导航网](https://chenzhuofan.top)  [https://chenzhuofan.top](https://chenzhuofan.top)