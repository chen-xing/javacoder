## java学习-springboot 注解介绍

>**郑重声明**: 本文首发于[人工博客](https://www.94rg.com)

![https://www.94rg.com](https://ae01.alicdn.com/kf/H7ffc2398a16c4f06b20531828ede40baD.png)





### 1、前言

> springboot 约定大于配置，内置了很多好用的高级功能。熟悉的掌握它们，一定能事半功倍。这里我将为大家介绍 Configuration、bean、ConditionalOnMissingBean、order、DependsOn的使用技巧与场景



### 2、Configuration

> 用@Configuration加载spring
> a、@Configuration配置spring并启动spring容器
> b、@Configuration启动容器+@Bean注册Bean
> c、@Configuration启动容器+@Component注册Bean
> c、使用 AnnotationConfigApplicationContext 注册 AppContext 类的两种方法
> d、映射配置文件



### 3、bean

> 直接把普通方法和对象注入到spring容器管理中
>
> @Bean
>
> public xxx aa(){}



### 4、ConditionalOnMissingBean

> 为自己的程序增加默认实现的时候这个非常有用。当用户为自定义的时候。注入这个缺省的bean



### 5、order

> 控制spring的启动顺序，ordery越小，优先级越高。默认优先级最低



### 6、DependsOn

> 当一个bean的实例化依赖于另一个bean的时候，它就出场了



### 7、给个示例

```
@Configuration
@Getter
public class DefenderConfiguration {

    @Value("${defenderUrlMapping}")
    private String defenderUrlMapping;

    @Autowired private DefenderUrlConfigurationAware defenderUrlConfigurationAware;

    @Bean
    @ConditionalOnMissingBean({DefenderUrlConfigurationAware.class})
    @DependsOn({"rgApplicationContextUtil"})
    public DefenderUrlConfigurationAware defenderUrlConfigurationAware(){
        return new DefenderUrlConfigurationAwareImpl();
    }

    @Bean
    @DependsOn({"defenderUrlConfigurationAware"})
    public UrlMappings urlMappings() {
        return new UrlMappings(defenderUrlConfigurationAware.getDefenderUrlMaps());
    }

    @Bean
    public FilterRegistrationBean defenderFilter() {

        FilterRegistrationBean registrationBean = new FilterRegistrationBean();
        registrationBean.setFilter(new DefenderFilter());
        registrationBean.setOrder(Integer.MIN_VALUE);
        registrationBean.addUrlPatterns("/*");
        return registrationBean;
    }



    @Bean(name="rgApplicationContextUtil")
    @Order()
    public RgApplicationContextUtil rgApplicationContextUtil() {
        RgApplicationContextUtil rgApplicationContextUtil = new RgApplicationContextUtil();
        return rgApplicationContextUtil;
    }


}
```







> 版权声明：本文为人工博客的原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接及本声明。
> 本文链接：[https://www.94rg.com/article/1739](https://www.94rg.com/article/1739)