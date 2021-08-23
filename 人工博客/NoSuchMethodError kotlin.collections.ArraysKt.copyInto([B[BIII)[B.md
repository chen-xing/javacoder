# NoSuchMethodError: kotlin.collections.ArraysKt.copyInto([B[BIII)[B

## 1、背景

升级七牛云客户端，附带把okhttp3给升级了一下到 4.5.0版本。
**结果是**

```
Caused by: java.lang.NoSuchMethodError: kotlin.collections.ArraysKt.copyInto([B[BIII)[B
	at okio.Segment.writeTo(Segment.kt:164)
	at okio.Segment.compact(Segment.kt:147)
	at okio.Buffer.write(Buffer.kt:1854)
	at okio.Buffer.read(Buffer.kt:1865)
```

<img src="https://oss.94rg.com/oneblog/20210714204606507.png-94rg002" width="500" height="300" align="middle" alt="郁闷了"/>


## 2、问题原因

okhttp3 4.x版本采用了kotlin进行了底层重写，所以如果你的项目是纯 Java 项目，需要添加添加 kotlin 的依赖



## 3、解决方案

### 3.1、引入kotlin依赖

```
<dependency>
    <groupId>org.jetbrains.kotlin</groupId>
    <artifactId>kotlin-stdlib</artifactId>
    <version>1.3.70</version>
</dependency>
若版本1.3.70不行的话可以改成1.3.50进行重试

或
org.jetbrains.kotlin:kotlin-stdlib:1.3.50
org.jetbrains.kotlin:kotlin-stdlib-common:1.3.50
```



### 3.2、降低okhttp3的版本

```
 <dependency>
    <artifactId>okhttp</artifactId>
    <groupId>com.squareup.okhttp3</groupId>
    <version>4.2.0</version>
</dependency>
```

我这边的 项目，刚好换成4.2.0就好了，如果不行建议换成3.x的版本就ok。



## 4、NoSuchMethodError的原因分析

NoSuchMethodError的报错字面的意思找不到对应的方法。

一般原因有以下两个：

+ 依赖缺失
+ 同一个jar多版本冲突。（这个情况比较多）

