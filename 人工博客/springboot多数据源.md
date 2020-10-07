### 1、什么场景需要多数据源

+ 业务读写分离
+ 业务分库
+ 业务功能模块拆分多库



### 2、常见的多数据源的方案

+ 按照数据源分别把mapper和entity放到不同的package下，然后用两个数据源分别注册、扫描对应的package,独立的sessionfactoty
+ 基于aop动态的切换的数据源

  

### 3、本文重点介绍的是基于aop的方案

#### 3.1、原理介绍

+ DatabaseType列出所有的数据源的key+++key
+ DatabaseContextHolder是一个线程安全的DatabaseType容器，并提供了向其中设置和获取DatabaseType的方法
+ DynamicDataSource继承AbstractRoutingDataSource并重写其中的方法determineCurrentLookupKey()，在该方法中使用DatabaseContextHolder获取当前线程的DatabaseType
+ MyBatisConfig中生成2个数据源DataSource的bean+++value
+ MyBatisConfig中将1）和4）组成的key+value对写入到DynamicDataSource动态数据源的targetDataSources属性（当然，同时也会设置2个数据源其中的一个为DynamicDataSource的defaultTargetDataSource属性中）
+ 将DynamicDataSource作为primary数据源注入到SqlSessionFactory的dataSource属性中去，并且该dataSource作为transactionManager的入参来构造DataSourceTransactionManager
+ 使用的时候，在dao层或service层先使用DatabaseContextHolder设置将要使用的数据源key，然后再调用mapper层进行相应的操作，建议放在dao层去做（当然也可以使用spring aop+自定注解去做）
+ 注意：在mapper层进行操作的时候，会先调用determineCurrentLookupKey()方法获取一个数据源（获取数据源：先根据设置去targetDataSources中去找，若没有，则选择defaultTargetDataSource），之后在进行数据库操作。



#### 3.2、代码示例

##### a、配置文件

```
spring.aop.proxy+target+class = true
spring.aop.auto = true
spring.datasource.druid.db1.url = 
spring.datasource.druid.db1.username = 
spring.datasource.druid.db1.password = 
spring.datasource.druid.db1.driver+class+name = com.mysql.jdbc.Driver
spring.datasource.druid.db1.initialSize = 5
spring.datasource.druid.db1.minIdle = 5
spring.datasource.druid.db1.maxActive = 20
spring.datasource.druid.db2.url = 
spring.datasource.druid.db2.username = 
spring.datasource.druid.db2.password = 
spring.datasource.druid.db2.driver+class+name = com.mysql.jdbc.Driver
spring.datasource.druid.db2.initialSize = 5
spring.datasource.druid.db2.minIdle = 5
spring.datasource.druid.db2.maxActive = 20
spring.datasource.druid.db3.url = 
spring.datasource.druid.db3.username = 
spring.datasource.druid.db3.password = 
spring.datasource.druid.db3.driver+class+name = com.mysql.jdbc.Driver
spring.datasource.druid.db3.initialSize = 5
spring.datasource.druid.db3.minIdle = 5
spring.datasource.druid.db3.maxActive = 20
```



##### b、生成Datasource

```
@Bean(name = "db1")
  @ConfigurationProperties(prefix = "spring.datasource.druid.db1")
  public DataSource db1() {
    return DruidDataSourceBuilder.create().build();
  }

  @Bean(name = "db2")
  @ConfigurationProperties(prefix = "spring.datasource.druid.db2")
  public DataSource db2() {
    return DruidDataSourceBuilder.create().build();
  }

  @Bean(name = "db3")
  @ConfigurationProperties(prefix = "spring.datasource.druid.db3")
  public DataSource db3() {
    return DruidDataSourceBuilder.create().build();
  }
```



##### c、定义数据源的key

```
@Getter
@AllArgsConstructor
public enum DBTypeEnum {
  db1("db1"),
  db2("db2"),
  db3("db3");
  private String value;
}
```



##### d、构造数据源和sessionFactory

```
/**
   * 动态数据源配置
   *
   * @return
   */
  @Bean
  @Primary
  public DataSource multipleDataSource(
      @Qualifier("db1") DataSource db1,
      @Qualifier("db2") DataSource db2,
      @Qualifier("db3") DataSource db3) {
    DynamicDataSource dynamicDataSource = new DynamicDataSource();
    Map<Object, Object> targetDataSources = new HashMap<>();
    targetDataSources.put(DBTypeEnum.db1.getValue(), db1);
    targetDataSources.put(DBTypeEnum.db2.getValue(), db2);
    targetDataSources.put(DBTypeEnum.db3.getValue(), db3);
    dynamicDataSource.setTargetDataSources(targetDataSources);
    dynamicDataSource.setDefaultTargetDataSource(db2);
    return dynamicDataSource;
  }

  @Bean("sqlSessionFactory")
  public SqlSessionFactory sqlSessionFactory() throws Exception {
    MybatisSqlSessionFactoryBean sqlSessionFactory = new MybatisSqlSessionFactoryBean();
    sqlSessionFactory.setDataSource(multipleDataSource(db1(), db2(), db3()));

    MybatisConfiguration configuration = new MybatisConfiguration();
    configuration.setJdbcTypeForNull(JdbcType.NULL);
    configuration.setMapUnderscoreToCamelCase(true);
    configuration.setCacheEnabled(false);
    sqlSessionFactory.setConfiguration(configuration);
    // PerformanceInterceptor(),OptimisticLockerInterceptor()
    // 添加分页功能
    sqlSessionFactory.setPlugins(new Interceptor[] {paginationInterceptor()});
            sqlSessionFactory.setGlobalConfig(globalConfiguration());
    return sqlSessionFactory.getObject();
  }
```



##### e、重写datasource切换策略

```
public class DynamicDataSource extends AbstractRoutingDataSource {

    @Override
    protected Object determineCurrentLookupKey() {
        return  DbContextHolder.getDbType();
    }
}
```



##### f、保存数据源切换的上下文信息

```
public class DbContextHolder {

  private static final ThreadLocal contextHolder = new ThreadLocal<>();
  /**
   * 设置数据源
   *
   * @param dbTypeEnum
   */
  public static void setDbType(DBTypeEnum dbTypeEnum) {
    contextHolder.set(dbTypeEnum.getValue());
  }

  /**
   * 取得当前数据源
   *
   * @return
   */
  public static String getDbType() {
    return (String) contextHolder.get();
  }

  /** 清除上下文数据 */
  public static void clearDbType() {
    contextHolder.remove();
  }
}
```



##### g、aop实现动态的数据源切换

```
@Component
@Order(value = +100)
@Slf4j
@Aspect
public class DataSourceSwitchAspect {

  @Pointcut("execution(* top.zhuofan.datafly.mapper.db1..*.*(..))")
  private void db1Aspect() {}

  @Pointcut("execution(* top.zhuofan.datafly.mapper.db2..*.*(..))")
  private void db2Aspect() {}

  @Pointcut("execution(* top.zhuofan.datafly.mapper.db3..*.*(..))")
  private void db3Aspect() {}

  @Before("db1Aspect()")
  public void db1() {
    log.debug("切换到db1 数据源...");
    DbContextHolder.setDbType(DBTypeEnum.db1);
  }

  @Before("db2Aspect()")
  public void db2() {
    log.debug("切换到db2 数据源...");
    DbContextHolder.setDbType(DBTypeEnum.db2);
  }

  @Before("db3Aspect()")
  public void db3() {
    log.debug("切换到db3 数据源...");
    DbContextHolder.setDbType(DBTypeEnum.db3);
  }
}
```



![人工博客](http://oss.94rg.com/oneblog/20200314112023114.jpg+94rg002)

#### 4、后续

更多精彩，敬请关注， [程序员导航网](https://chenzhuofan.top/) [https://chenzhuofan.top](https://chenzhuofan.top/)



版权声明：本文为人工博客的原创文章，遵循 CC 4.0 BY+SA 版权协议，转载请附上原文出处链接及本声明。
本文链接：https://www.94rg.com/article/8