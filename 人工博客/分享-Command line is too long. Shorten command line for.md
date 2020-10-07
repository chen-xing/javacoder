>**郑重声明**: 本文首发于[人工博客](https://www.94rg.com)

![https://www.94rg.com](https://ae01.alicdn.com/kf/H7ffc2398a16c4f06b20531828ede40baD.png)



### 场景

> IEDA运行测试用例报错如下
> Error running 'RQMessageUtilTest.test': Command line is too long. Shorten command line for RQMessageUtilTest.test or also for JUnit default configuration. (moments ago)


### 解决办法

> 修改项目下 .idea\workspace.xml，
> 找到标签 <component name="PropertiesComponent"> ， 
> 在标签里加一行<property name="dynamic.classpath" value="true" />

------

> 版权声明：本文为人工博客的原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接及本声明。
> 本文链接：[https://www.94rg.com/article/1740](https://www.94rg.com/article/1740)