### java spi入门

### 摘要

SPI全称Service Provider Interface，是Java提供的一套用来被第三方实现或者扩展的接口，它可以用来启用框架扩展和替换组件。 SPI的作用就是为这些被扩展的API寻找服务实现。

### 关键字

spi，插件机制



### 1、什么是SPI

​	SPI全称Service Provider Interface，是Java提供的一套用来被第三方实现或者扩展的接口，它可以用来启用框架扩展和替换组件。 SPI的作用就是为这些被扩展的API寻找服务实现。

### 2、SPI和API的区别

+ spi **Service Provider Interface** `调用方`来制定接口，`实现方`来针对接口来实现不同的实现。`调用方`来选择自己需要的`实现方`。
+ API Application Programming Interface
  大多数情况下，都是实现方来制定接口并完成对接口的不同实现，调用方仅仅依赖却无权选择不同实现。

[![SPI与API的区别](https://s3.ax1x.com/2020/11/14/DCEICq.png)](https://imgchr.com/i/DCEICq)

### 3、SPI的使用场景

+ JDBC加载不同类型数据库的驱动
+ SLF4J加载不同提供商的日志实现类
+ Spring中大量使用了SPI,比如：对servlet3.0规范对ServletContainerInitializer的实现、自动类型转换Type Conversion SPI(Converter SPI、Formatter SPI)等
+ Dubbo中也大量使用SPI的方式实现框架的扩展, 不过它对Java提供的原生SPI做了封装，允许用户扩展实现Filter接口
+ skywailking插件机制
+ 常规的idea插件，皮肤插件

### 4、SPI如何使用

+ 实现服务提供者提供的接口，在jar包的META-INF/services目录下创建一个以“接口全限定名”为命名的文件，内容为实现类的全限定名；
+ 接口实现类所在的jar包放在主程序的classpath中；
+ 主程序通过java.util.ServiceLoder动态装载实现模块，它通过扫描META-INF/services目录下的配置文件找到实现类的全限定名，把类加载到JVM；
+ SPI的实现类必须携带一个不带参数的构造方法；

### 5、代码示例

**定义接口**

```
public interface HelloService {
    void hello();
}
```

**实现接口**

```
public class HelloService1Impl implements HelloService {
    @Override
    public void hello() {
        System.out.println("hello jiaboyan");
    }
}

public class HelloService2Impl implements HelloService {
    @Override
    public void hello() {
        System.out.println("hello world");
    }
}
```

**添加配置文件**
注意文件名必须是接口的全路径，配置文件放在对应的实现类的jar中

```
com.jiaboyan.test.impl.HelloService1Impl
com.jiaboyan.test.impl.HelloService2Impl
```


**调用逻辑**

```
public class Test {

    public static void main(String[] agrs) {
        ServiceLoader<HelloService> loaders = ServiceLoader.load(HelloService.class);
        for (HelloService helloService : loaders) {
            helloService.hello();
        }
    }
}
```



### 6、优缺点

+ 优点：代码解耦，不用硬编码

+ 缺点：实例化造成延时加载，不能按需加载；线程不安全

  