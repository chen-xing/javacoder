### java如何实现一个方法有多个返回值-java 元组-javatuples的使用

### 摘要

java实现一个方法有多个返回值的方式有：Set、自定义bean、java元组javatuples

### 关键字

多个返回值，javatuples

### 1、需求

java实现一个方法有多个返回值

+ Set(返回值类型必须一致)
+ 自定义bean (项目中过多的中间bean不利于阅读)
+ java 元组 javatuples (比较优雅的一种实现方式)



### 2、如何使用

#### 2.1、引入pom

```
 <dependency>
     <groupId>org.javatuples</groupId>
     <artifactId>javatuples</artifactId>
     <version>1.2</version>
 </dependency>
```



#### 2.2、api的简介

主要的api类型是按照返回值的个数进行定义的，比如 pair 、Triplet、Quartet。

分别应 2到4个返回值。这个东西的好处是参数是泛型实现的。

举个简单的使用例子：

```
Quartet<String, String, String, String> quartet= Quartet.with(a, b, c, d);
```

