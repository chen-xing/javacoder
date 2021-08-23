# postman实现从csv中读取参数循环调用

## 1、背景

有一批数据需要修正，从操作上看是借助postman，串行调用2个接口就能解决问题。问题待处理的数据比较多。那么postman是否支持csv的参数化?答案是肯定的。

## 2、csv文件准备

新建一个文件 1.csv

内容是:

```
flowId
cbcf4bf275544bad9f11f8dc6e780fdc
0d46db3e55164f52a7b3978892e6ea19
58e0309f151f48bcb17b1fbbf5338608
0c13aa4b11e14d1087e8adb04265f2ed
```

**第一列必须制定列名**，因为后面的参数化是通过这个列名来确定的。



## 3、准备request

![准备rest接口](https://cdn.jsdelivr.net/gh/chen-xing/figure_bed_02/cdn/20210722205000618.png)



tests中增加校验

> tests["Status code is 200"] **=** responseCode.code **===** 200



## 4、关联参数与接口

![postman runner](https://cdn.jsdelivr.net/gh/chen-xing/figure_bed_02/cdn/20210722205426054.png)

+ 选择对应接口上的collection，右键 run collection
+ 调出 runner设置窗口
+ 选择关联文件
+ 执行 run

## 5、查看执行的结果

![postman runner的结果](https://cdn.jsdelivr.net/gh/chen-xing/figure_bed_02/cdn/20210722205732145.png)