### 故障处理系列-httpclient 3.x的bug引发的惨案

### 1、故障现象描述

调用链路是 A->B->C

目前出故障的是B服务。A是网关，C是底层服务

故障的现象是：

+ A对外出现大量的504
+ B的线程数暴增
+ B的流量骤减，一段时间趋近于0
+ C的请求响应无明显异常

### 2、故障分析

从B的流量骤减至0，然后A大量504可以推测出可能是A把B给熔断了，根据这个思路排查，最终定位到了B的熔断策略中有一个最大线程数的限制，这个跟B的线程数暴增是吻合的。

但是C是ok的，而B的线程数据很高，一般都是死锁或者连接泄露的问题。

按照这思路排查下去，果不其然，发现了httpClient的一个大坑.



### 3、故障还原

结论是apache的commons-httpclient 3.x 存在连接泄露问题

#### 3.1 问题特征

- 无法从连接池中获取连接，异常描述为`timeout waiting for connection`
- `netstat`看到连接状态为`CLOSE_WAIT`
- 目标服务器`ping`得通，甚至`curl`等方式亦可以获取结果

```
try {
  int statuCode = client.executeMethod(method);
  if (statuCode == HttpStatus.SC_OK) {
    return method.getResponseBody();
  }
} catch (IOException e) {
  e.printStackTrace();
} finally {
  // required
  // method.releaseConnection();
}
```

问题位于第三行到第五行。判断了响应码之后再决定是否`getResponseBody`，这里会造成连接泄露。
假如你按照例程补上了`method.releaseConnection`，那就不会造成泄露



让我们使用例程代码来回答：

```
public static void main(String[] args) throws InterruptedException {
    HttpClientParams params = new HttpClientParams();
    params.setConnectionManagerClass(MultiThreadedHttpConnectionManager.class);
    HttpClient client = new HttpClient(params);
    getPage(client);
}
 
private static void getPage(HttpClient client) {
    GetMethod method = new GetMethod("http://xnnyygn.in");
    try {
        client.executeMethod(method);
        log.info("get response");
        method.getResponseBodyAsString();
    } catch (IOException e) {
        log.error("failed to get page", e);
    } finally {
        // method.releaseConnection();
    }
}
```

注意我把releaseConnection注释掉了。在开启了DEBUG级别日志之后，你可以看到日志中输出

```
17:28:15,999  DEBUG MultiThreadedHttpConnectionManager$ConnectionPool : Allocating new connection, hostConfig=HostConfiguration[host=http://xnnyygn.in]
17:28:16,003  DEBUG HttpConnection : Open connection to xnnyygn.in:80
17:28:19,866  DEBUG HttpMethodBase : Adding Host request header
17:28:20,063  INFO  HttpClientTest : get response
17:28:20,064  DEBUG HttpMethodBase : Buffering response body
17:28:20,067  DEBUG HttpMethodBase : Should NOT close connection in response to directive: keep-alive
17:28:20,067  DEBUG HttpConnection : Releasing connection back to connection manager.
17:28:20,068  DEBUG MultiThreadedHttpConnectionManager$ConnectionPool : Freeing connection, hostConfig=HostConfiguration[host=http://xnnyygn.in]
```

注意get response的位置，和之后的`Releasing connection back to connection manager`。代码中没有显示调用releaseConnection，但是连接被回收了。简单来说，这是`getResponseBody`和关联方法比如`getResponseBodyAsString`本身的特性。理论上获取了全部响应之后，你不太可能再从socket的input中读取数据了，所以connection的使命也完成了，可以销毁或者回收进入连接池了。你可以阅读httpclient的源码了解这一逻辑。另外还有根据rfc2612(http)不可能有body的响应码，比如304，遇到这种响应码，在executeMethod之后你的connection就被回收了。这些逻辑是httpclient内部在最小化connection的使用，尽快结束重型资源的使用的设计。但是这不代表你可以依赖这种特性。

另一方面，当你没有调用`getResponseBody`时，httpclient无法帮你关闭connection，必须你手动`releaseConnection`。这也是造成第二段代码中前10次请求没有关闭connection导致连接池被占满，第11次请求无法获取连接的原因。

到这里，相信大部分人应该都明白了。最开始的代码在不知道的情况下依赖了自动关闭connection这个特性，但是遇到非200的响应码时就没辙了。解决方法就是加上`method.releaseConnection`，就如例程一样。

小结一下，虽然是大家平时一直使用的httpclient，但是很多人还是存在误用。在你不了解的情况，务必严格按照例程的方式写代码，否则容易造成各种诡异的问题。这里是搭配 MultiThreadedHttpConnectionManager出了问题的。

### 4、改进措施

+ 显式释放httpConnection
+ 增加获取http连接的超时时间 setConnectionManagerTimeout
+ 其他的各种超时时间最好也根据业务特性进行明确设置
+ 升级版本到最新的版本或直接切换到okhttp3

代码示例

```
HttpConnectionManager httpConnectionManager = new MultiThreadedHttpConnectionManager();
HttpConnectionManagerParams params = httpConnectionManager.getParams();
params.setConnectionTimeout(5000);
params.setSoTimeout(18000);
params.setDefaultMaxConnectionsPerHost(1000);
params.setMaxTotalConnections(2000);
client = new HttpClient(httpConnectionManager);
client.getParams().setContentCharset(UTF8);
client.getParams().setHttpElementCharset(UTF8);
client.getParams().setConnectionManagerTimeout(5000);

try {
  int statuCode = client.executeMethod(method);
  if (statuCode == HttpStatus.SC_OK) {
    return method.getResponseBody();
  }
} catch (IOException e) {
  e.printStackTrace();
} finally {
  method.releaseConnection();
}

```



