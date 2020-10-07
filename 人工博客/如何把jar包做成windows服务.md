# 如何把jar包做成windows服务

#### 1、在idea中用maven将程序打成jar，放到运行的目录中。

#### 2、去github上面下载winsw： https://github.com/kohsuke/winsw/releases

![dHte4x.png](https://s1.ax1x.com/2020/08/29/dHte4x.png)

#### 3、将WinSW.NET4.exe文件复制到java程序所在文件夹中

#### 4、将java程序重命名，去掉名称中的“.”。例如HelloWorld-1.0.jar  ---->  HelloWorld.jar

#### 5、将WinSW.exe重命名为HelloWorld.exe(和jar同名)

#### 6、新建一个xml文件，命名为HelloWorld.xml，写入以下内容（还有一些参数自己去看github说明）

```
<service>

     <id>HelloWorld</id>

     <name>HelloWorld</name>

     <description>This is HelloWorld service.</description>

     <!-- java环境变量 -->

     <env name="JAVA_HOME" value="%JAVA_HOME%"/>

     <executable>java</executable>

    <arguments>-jar "E:\springboot\ HelloWorld.jar"</arguments>

     <!-- 开机启动 -->

     <startmode>Automatic</startmode>

     <!-- 日志配置 -->

     <logpath>%BASE%\log</logpath>

     <logmode>rotate</logmode>

 </service>

```

如果没有配置环境变量，直接将三个文件扔到java的bin目录下运行。去掉标签<env name="JAVA_HOME"           value="%JAVA_HOME%"/>

#### 7、命令行定位到当前目录，执行： 

```
 test.exe  install
```



#### 8、去windows服务列表中启动程序

（如果需要更新程序，只需要先将服务停止，再将新文件重命名为HelloWorld.jar，最后启动服务就行了）



#### 9、**更高效的方法**

1.去nssm[官网](http://www.nssm.cc/release/nssm-2.24.zip)下载你nssm工具

2.解压文件，选择64/32位程序打开，就会看到如下图示：

[![dHUNjS.png](https://s1.ax1x.com/2020/08/29/dHUNjS.png)](https://imgchr.com/i/dHUNjS)

界面显示很直接明了，一般输入这两个参数即可，有其他需要可自己配置



