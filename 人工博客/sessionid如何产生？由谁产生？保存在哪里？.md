# sessionid如何产生？由谁产生？保存在哪里？

## 1、sessionid是什么

sessionid是一个会话的key，浏览器第一次访问服务器会在服务器端生成一个session，有一个sessionid和它对应。

tomcat生成的sessionid叫做jsessionid。

session在访问tomcat 服务器HttpServletRequest的getSession(true)的时候 创建，tomcat的ManagerBase类提供创建sessionid的方法： 随机数+时间+jvmid；

## 2、存储在哪里

存储在服务器的内存中，tomcat的StandardManager类将session 存储在内存中，也可以持久化到file，数据库，memcache，redis等。 客户端只保存sessionid到cookie中，而不会保存session，session销毁只能通过invalidate或超时，关掉浏览器并不会关闭session。



## 3、何时创建

那么Session在何时创建呢？当然还是在服务器端程序运行的过程中创建的，

不同语言实现的应用程序有不同创建Session的方法，而在Java中是通过调用HttpServletRequest的getSession方法（使用true作为参数）创建的。在创建了Session的同时，服务器会为该Session生成唯一的Session id，而这个Session id在随后的请求中会被用来重新获得已经创建的Session；在Session被创建之后，就可以调用Session相关的方法往Session中增加内容了，而这些内容只会保存在服务器中，发到客户端的只有Session id；当客户端再次发送请求的时候，会将这个Session id带上，服务器接受到请求之后就会依据Session id找到相应的Session，从而再次使用之。

+ 创建：sessionid第一次产生是在直到某server端程序调用 HttpServletRequest.getSession(true)这样的语句时才被创建。

+ 删除：超时；程序调用HttpSession.invalidate()；程序关闭；

session存放在哪里：服务器端的内存中。不过session可以通过特殊的方式做持久化管理（memcache，redis）。

session的id是从哪里来的，sessionID是如何使用的：当客户端第一次请求session对象时候，服务器会为客户端创建一个session，并将通过特殊算法算出一个session的ID，用来标识该session对象

session会因为浏览器的关闭而删除吗？

不会，session只会通过上面提到的方式去关闭。

 

## 4、tomcat中session的创建：

ManagerBase是所有session管理工具类的基类，它是一个抽象类，所有具体实现session管理功能的类都要继承这个类，该类有一个受保护的方法，该方法就是创建sessionId值的方法：

（ tomcat的session的id值生成的机制是一个随机数加时间加上jvm的id值，jvm的id值会根据服务器的硬件信息计算得来，因此不同jvm的id值都是唯一的），

StandardManager类是tomcat容器里默认的session管理实现类，

它会将session的信息存储到web容器所在服务器的内存里。

PersistentManagerBase也是继承ManagerBase类，它是所有持久化存储session信息的基类，PersistentManager继承了PersistentManagerBase，但是这个类只是多了一个静态变量和一个getName方法，目前看来意义不大， 对于持久化存储session，tomcat还提供了StoreBase的抽象类，它是所有持久化存储session的基类，另外tomcat还给出了文件存储FileStore和数据存储JDBCStore两个实现。

浏览器禁用cookie后，怎么使用session，解决方案:
sessionid是存储在cookie中的，解决方案如下：

Session URL重写，保证在客户端禁用或不支持COOKIE时，仍然可以使用Session

session机制。session机制是一种服务器端的机制，服务器使用一种类似于散列表的结构（也可能就是使用散列表）来保存信息。

当程序需要为某个客户端的请求创建一个session时，服务器首先检查这个客户端的请求里是否已包含了一个session标识（称为session id），如果已包含则说明以前已经为此客户端创建过session，服务器就按照session id把这个session检索出来使用（检索不到，会新建一个），如果客户端请求不包含session id，则为此客户端创建一个session并且生成一个与此session相关联的session id，session id的值应该是一个既不会重复，又不容易被找到规律以仿造的字符串，这个session id将被在本次响应中返回给客户端保存。 保存这个session id的方式可以采用cookie，这样在交互过程中浏览器可以自动的按照规则把这个标识发挥给服务器。一般这个cookie的名字都是类似于 SEEESIONID。但cookie可以被人为的禁止，则必须有其他机制以便在cookie被禁止时仍然能够把session id传递回服务器。 经常被使用的一种技术叫做URL重写，就是把session id直接附加在URL路径的后面。还有一种技术叫做表单隐藏字段。就是服务器会自动修改表单，添加一个隐藏字段，以便在表单提交时能够把session id传递回服务器。比如：
```
<form name=”"testform”" action=”"/xxx”"> 
    <input type=”"hidden”" name=”"jsessionid”" value=”"ByOK3vjFD75aPnrF7C2HmdnV6QZcEbzWoWiBYEnLerjQ99zWpBng!-145788764″”>
    <input type=”"text”"> 
</form>
```
URL重写：

http://www.test.com/test;jsessionid=ByOK3vjFD75aPnrF7C2HmdnV6QZcEbzWoWiBYEnLerjQ99zWpBng!-145788764

##  5、session集群

目前，为了使web能适应大规模的访问,需要实现应用的集群部署. 而实现集群部署首先要解决session的统一，**即需要实现session的共享机制**。

 目前，在集群系统下实现session统一的有如下几种方案：

(1) 应用服务器间的session复制共享（如tomcat session共享）
(2) 基于cache DB缓存的session共享



### 5.1、应用服务器间的session复制共享

       session复制共享，主要是指集群环境下，多台应用服务器之间同步session，使session保持一致，对外透明。 如果其中一台服务器发生故障，根据负载均衡的原理，web服务器（apache/nginx）会遍历寻找可用节点，分发请求，由于session已同步， 故能保证用户的session信息不会丢失。

 












此方案的不足之处：

+ 技术复杂,必须在同一种中间件之间完成(如:tomcat-tomcat之间).+session复制带来的性能损失会快速增加.特别是当session中保存了较大的对象,而且对象变化较快时, 性能下降更加显著. 这种特性使得web应用的水平扩展受到了限制。
+   Session内容序列化（serialize），会消耗系统性能。
+   Session内容通过广播同步给成员，会造成网络流量瓶颈，即便是内网瓶颈。  

### 5.2、  基于cache DB缓存的session共享

       即使用cacheDB存取session信息，应用服务器接受新请求将session信息保存在cache DB中，当应用服务器发生故障时，web服务器（apache/nginx）会遍历寻找可用节点，分发请求，当应用服务器发现session不在本机内存 时，则去cache DB中查找，如果找到则复制到本机，这样实现session共享和高可用。



目前有开源的msm用于解决tomcat之间的session共享：Memcached_Session_Manager（MSM）

http://code.google.com/p/memcached-session-manager/


一个高可用的Tomcat session共享解决方案，除了可以从本机内存快速读取Session信息(仅针对黏性Session)外，同时可使用memcached存取Session，以实现高可用。

 

特性

+ 支持Tomcat6、Tomcat7支持黏性、非黏性Session
+ 无单一故障点
+ 可处理tomcat故障转移
+ 可处理memcached故障转移
+ 插件式session序列化
+ 允许异步保存session，以提升响应速度
+ 只有当session有修改时，才会将session写回memcached
+ JMX管理&监控

该方案的不足之处：

+ memcache支持的数据结构比较单一
+ memcache的内存必须足够大，否则会出现用户session从Cache中被清除
+ 需要定期的刷新缓存
+ 服务器故障时，存在于内存的memcache数据将会丢失
+ 为了解决基于memcache中存在的不足，故提出了下面的一种解决方案：

 

### 5.3、基于redis缓存的session共享

结合上面的 MSM 思想，由 redis负责 session 数据的存储，而我们自己实现的 session manager 将负责 session 生命周期的管理。

+ 此架构存在着当redis master故障时, 虽然可以有一到多个备用slave，但是redis不会主动的进行master切换，这时session服务中断为了做到redis的高可用，引入了zookper或者haproxy或者keepalived来解决redis master slave的切换问题。即：此体系结构中, redis master出现故障时, 通过haproxy设置redis slave为临时master, redis master重新恢复后, 再切换回去. 此方案中, redis-master 与redis-slave 是双向同步的, 解决目前redis单点问题. 这样保证了session信息



在redis中的高可用。

实现此方案：

+   nginx        1   192.168.1.102

+   tomcat1    1  

+   tomcat2    1

+   redis-master   1 

+   redis-slave      1

+   slave1     1

+   slave2     1

+   haproxy vip  1