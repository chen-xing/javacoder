# DataSource和Transactional原理介绍

## 1、基础概念

| 名词        | 介绍                                                         |
| ----------- | ------------------------------------------------------------ |
| jdbc        | java操作数据库的一个规范                                     |
| connection  | java程序与数据库建立的网络连接，是操作数据库的核心。但是创建和销毁是比较耗费资源 |
| datasource  | 池化复用connection，提供程序的性能                           |
| transaction | 保证一组相关联的数据库操作的一致性，要么同时成功，要么同时失败 |



## 2、工作原理

### 2.1、dataSource的原理

利用池化技术，维护了一定量的connection.减少了创建和销毁connection带来的性能损耗。同时可以提供一些辅助的功能，如sql预编译、sql监控等

### 2.2、transaction的原理

要保证事务，核心是要保证所有的sql的执行都使用的是同一个connection，且必须是手动提交的模式。所以当是transaction的模式下，强制将autoCommit修改为false,将connection以ThreadLocal的模式进行了存储，保障了在同一个线程的同一个事务里的获取的connection都是同一个。

## 3、容易出现的问题

### 3.1、连接池管理

连接池管理关键的2个参数是必须设置**最大的等待时间**和**最大的等待线程数**

### 3.2、事务模式下无法切换数据源

事务模式下，内部切换数据源是无效的，原因是内部获取的conection都是从threadlocal中获取的。都是事务内第一个获取的链接。

案例细节参考：[**@Transactional导致AbstractRoutingDataSource动态数据源无法切换的解决办法**](https://www.94rg.com/article/1847)