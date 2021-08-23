# @Transactional导致AbstractRoutingDataSource动态数据源无法切换的解决办法

## 1、背景

项目的数据库采用了读写分离多数据源，采用AOP进行拦截，利用ThreadLocal及AbstractRoutingDataSource进行数据源切换，数据源代码如下:

```javascript
public class RoutingDataSource extends AbstractRoutingDataSource {

    @Override
    protected Object determineCurrentLookupKey() {
        return DBContext.getDBKey();
    }
}
```

AOP细节就不讲了，大致是拦截mybatis的Mapper层，约定对方法前缀，比如update/delete/insert/save开头的认为是写方法，切换到主库，其它方法切换到从库。spring的xml配置如下：

数据源：

```javascript
 1     <bean id="dsAlfred" class="cn.mwee.utils.datasource.RoutingDataSource">
 2         <property name="targetDataSources">
 3             <map key-type="java.lang.String">
 4                 <entry key="master" value-ref="dsAlfred_master"/>
 5                 <entry key="slave1" value-ref="dsAlfred_slave1"/>
 6                 <entry key="slave2" value-ref="dsAlfred_slave2"/>
 7                 <entry key="history" value-ref="dsAlfred_history"/>
 8             </map>
 9         </property>
10         <property name="defaultTargetDataSource" ref="dsAlfred_master"/>
11     </bean>
```

事务部分：

```javascript
1     <bean id="alfredTxManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
2         <property name="dataSource" ref="dsAlfred"/>
3     </bean>
4     <tx:annotation-driven transaction-manager="alfredTxManager"/>
```

一直用了很久，都很正常(不管是事务方法，还是非事务方法)，最近几天发现有一个服务，更新数据库时，一直报read-only异常，当时判断应该是连接到从库上了(注：从库是只读权限，无法更新数据)，方法伪代码如下：

```javascript
1 @Transactional
2 void doSomeThing(){
3 　　xxxMapper.select(...);
4    yyyMapper.update(...);
5    ...
6 }　
```

执行到第4行的时候，死活切换不到master主库上来，哪怕在doSomeThing方法的首行，设置DBContext.setDBKey("master") 都不好使，而其它类似的方法都正常。于是对比了代码，发现这个方法被调用的地方，最近加了几行代码，伪代码如下：

```javascript
    public void method1(){
        xxxMapper.select(...);
        ...
        doSomeThing();
    }
```

即：在调用doSomeThing()方法前，最近因为需求变更，前面加了一行查询操作（大家不用纠结为啥加这一行，产品需要~_~)，把这个查询去掉，再执行，就ok了，然后... 然后就开始思考人生了...

## 2、原因

各种百度，google后，最后在org.springframework.jdbc.datasource.DataSourceTransactionManager#doBegin 这个类的源代码中找到了答案：

```javascript
 1 @Override
 2     protected void doBegin(Object transaction, TransactionDefinition definition) {
 3         DataSourceTransactionObject txObject = (DataSourceTransactionObject) transaction;
 4         Connection con = null;
 5 
 6         try {
 7             if (txObject.getConnectionHolder() == null ||
 8                     txObject.getConnectionHolder().isSynchronizedWithTransaction()) {
 9                 Connection newCon = this.dataSource.getConnection();
10                 if (logger.isDebugEnabled()) {
11                     logger.debug("Acquired Connection [" + newCon + "] for JDBC transaction");
12                 }
13                 txObject.setConnectionHolder(new ConnectionHolder(newCon), true);
14             }
15 
16             txObject.getConnectionHolder().setSynchronizedWithTransaction(true);
17             con = txObject.getConnectionHolder().getConnection();
18 
19             Integer previousIsolationLevel = DataSourceUtils.prepareConnectionForTransaction(con, definition);
20             txObject.setPreviousIsolationLevel(previousIsolationLevel);
21 
22             // Switch to manual commit if necessary. This is very expensive in some JDBC drivers,
23             // so we don't want to do it unnecessarily (for example if we've explicitly
24             // configured the connection pool to set it already).
25             if (con.getAutoCommit()) {
26                 txObject.setMustRestoreAutoCommit(true);
27                 if (logger.isDebugEnabled()) {
28                     logger.debug("Switching JDBC Connection [" + con + "] to manual commit");
29                 }
30                 con.setAutoCommit(false);
31             }
32 
33             prepareTransactionalConnection(con, definition);
34             txObject.getConnectionHolder().setTransactionActive(true);
35 
36             int timeout = determineTimeout(definition);
37             if (timeout != TransactionDefinition.TIMEOUT_DEFAULT) {
38                 txObject.getConnectionHolder().setTimeoutInSeconds(timeout);
39             }
40 
41             // Bind the connection holder to the thread.
42             if (txObject.isNewConnectionHolder()) {
43                 TransactionSynchronizationManager.bindResource(getDataSource(), txObject.getConnectionHolder());
44             }
45         }
46 
47         catch (Throwable ex) {
48             if (txObject.isNewConnectionHolder()) {
49                 DataSourceUtils.releaseConnection(con, this.dataSource);
50                 txObject.setConnectionHolder(null, false);
51             }
52             throw new CannotCreateTransactionException("Could not open JDBC Connection for transaction", ex);
53         }
54     }
```

注意：第7-16行，在开始一个事务前，如果当前上下文的连接对象为空，获取一个连接对象，然后保存起来，下次doBegin再调用时，就直接用这个连接了，根本不做任何切换(类似于缓存命中！）

这样就解释得通了： doSomeThing()方法被调用前，加了一段select方法，相当于已经切换到了slave从库，然后再进入doBegin方法时，就直接拿这个从库的链接了，不再进行切换。那为啥其它同样启用事务的方法，又能正常连到主库呢？同样的解释，因为这类方法前面，没有任何其它操作，而xml中的动态数据源配置，默认连接的就是master主库，因此没有问题。

弄明白了之后，解决办法自然就有了：

```javascript
    public void method1(){
        DBContext.setDBKey("master");//先切换到主库
        xxxMapper.select(...);
        ...
        doSomeThing();
    }
```

先切到主库上来，这样后面再调用有事务的方法时，就仍然保持在主库的连接上。 