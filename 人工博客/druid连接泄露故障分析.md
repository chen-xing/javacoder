# druid连接泄露故障分析

## 1、问题的如何发生的

### 1.1、应用功能介绍

+ 系统是一个双数据源双写单独的服务。(两个数据源是不同的存储，所以无法使用主从复制的模式，是一个切换存储介质的过渡态)。
+ 历史代码有个更新逻辑update xx set a=b where m=n。但是这个表中的记录超10亿。遇到需要更新的记录比较多的场景下存在问题。故对这个进行了sql优化。采用的逻辑是查询出需要更新的记录id，然后分页更新。

### 1.2、关键代码

**双数据源操作**

```
private Object runSql(List<String> sqlSessionFactotyBeanNameList, MethodInvocation invocation)
            throws InvocationTargetException, IllegalAccessException {
        List<SqlSession> sqlSessionList = Lists.newArrayList();
        Object result = null;
        try {
            for (String sessionFactotyBeanName : sqlSessionFactotyBeanNameList) {
                SqlSessionFactory sqlSessionFactory =
                        RgApplicationContextUtil.getBean(
                                sessionFactotyBeanName, SqlSessionFactory.class);
                SqlSession sqlSession = sqlSessionFactory.openSession(ExecutorType.BATCH);
                Object mapper = sqlSession.getMapper(invocation.getMethod().getDeclaringClass());
                Object[] param = invocation.getArguments();
                result = invocation.getMethod().invoke(mapper, param);
                sqlSessionList.add(sqlSession);//问题代码，注意!!!!
                sqlSession.commit();
            }
        } catch (Exception ex) {
            sqlSessionList.stream()
                    .forEach(
                            x -> {
                                x.rollback();
                            });
        } finally {
            sqlSessionList.stream()
                    .forEach(
                            x -> {
                                x.close();
                            });
        }
        return result;
    }
```

**问题的sql**

```
<select id="getBatchIdWithLimit" resultType="java.lang.Long">
	SELECT x.id FROM context x WHERE x.oid = #{oid} ORDER BY id ASC
	LIMIT #{offset}, #{limit}
</select>
```

**关键的配置**

> maxWait  获取连接时最大等待时间，单位毫秒。配置了maxWait之后，缺省启用公平锁，并发效率会有所下降，如果需要可以通过配置useUnfairLock属性为true使用非公平锁。
>
> 当前系统此参数未进行配置，所以会无限等待，使用的是公平锁

### 1.3、问题出现的步骤

+ sql中存在问题，部分数据的长度超过Integer的最大值（2147483647），映射存在问题。
+ 双数据源代码存在bug。 List<SqlSession>的代码结合的 add 位置过于落后，导致反射出现异常的时候。当次的SqlSession未关联到待处理的集合中，进而也就未rollback和close。造成链接泄露。
+ 当出现问题的数据的时候，结合双数据源的代码的bug。会造成List<SqlSession>为空，所以未进行释放操作，(链接泄露了)
+ 当前系统最大的连接数是100，出现了100次这样的数据，这个服务就回无尽的等待获取链接中的状态。

### 1.4、问题的表象

![image-20210601165753462](https://cdn.jsdelivr.net/gh/chen-xing/figure_bed/images/20210601165802.png)



![代码的问题点](https://cdn.jsdelivr.net/gh/chen-xing/figure_bed/images/20210601204828.png)

## 2、如何复现问题

### 2.1、问题数据复现

+ 把数据库的最大连接数调整成1，maxWaitTime不设置
+ 构造一条id大于2147483647的数据
+ 使用api 触发调用到这个逻辑
+ 结果是:第一次调用报错，第二次调用会卡的客户端设置的超时时间。

### 2.2、数据库连接异常复现

> 还有一种路径是代码都没问题，但是由于高并发造成数据库是锁。mybatis是可以设置sql的执行时长的。一旦出现了这种场景。问题也是会出现的。
>
> 但是这种场景比较难以复现，那么有没有一种手段可以高效的伪造这个场景。



**准备知识**

```
set autocommit=0;　　//关闭数据的事务自动提交
SELECT * FROM xxx a WHERE a.id='111' for update; //获取数据库的行锁
commit;//提交事务
```

> 数据默认是自动提交的，所以前置set autocommit=0;这个操作不要忘记了，踩过几次坑。完成后执行commit;进行解锁。
>
> 测试完毕记得set autocommit=1;来恢复数据库的事务自动提交的特性。

+ 准备一条接口测试用的数据
+ 执行sql  select ..for update 进行行记录锁定
+ 接口调用使用同一个id进行请求。因为记录锁定了，所以api的更新是失败了，成功的伪造了高并发形成了行锁造成的sql问题

## 3、问题总结

+ 数据库的保护配置:maxActive、maxWait都配置上，相当于熔断保护
+ mybatis对象映射需要关注数据的范围
+ 利用select for update制造行锁伪造高并发造成的数据问题
