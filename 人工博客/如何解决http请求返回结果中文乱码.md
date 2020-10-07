## 如何解决http请求返回结果中文乱码

### 1、问题描述

```
http请求中，请求的结果集中包含中文，最终以乱码展示。
```

### 2、问题的本质

```
乱码的本质是服务端返回的字符集编码与客户端的编码方式不一致。
场景的如服务端返回 ISO-8859-1，而客户端的编码默认是UTF-8
```

###  3、解决办法

```
解决的办法就是让服务端返回的结果的编码与客户端的编码保持一致
最直接有效的方法是在request的header中增加一个项
Accept:application/json;charset=UTF-8
```

### 4、题外拓展

```
http中最常见的2个header的区别
Content-Type:application/json;charset=UTF-8
Accept:application/json;charset=UTF-8

Content-Type 用于描述本次请求的body的内容是json格式，且编码为UTF-8
Accept 用于描述客户端希望返回的结果以json来组织，且UTF-8
Content-Type 用于描述request,而Accept用于描述reponse
```



### 5、后续

```
更多精彩，敬请关注， [ 程序员导航网](https://chenzhuofan.top)  [https://chenzhuofan.top](https://chenzhuofan.top)
```



