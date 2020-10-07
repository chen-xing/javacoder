##1.使用maven构建多模块工程
#####第一步:mvn archetype:generate -DgroupId=com.ttyl -DartifactId=ttyl_parent -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false -DarchetypeCatalog=local -X

*注意 mvn archetype:create 已经不可用

####第二步:将src文件夹删除，然后修改pom.xml文件，将<packaging>jar</packaging>修改为<packaging>pom</packaging>，pom表示它是一个被继承的模块
####第三步:mvn archetype:generate -DgroupId=com.ttyl -DartifactId=ttyl_utils -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false -DarchetypeCatalog=local -X
####第四步: 执行mvn idea:idea 将项目转化成idea项目
####第五步:用idea打开项目

****
解决生成慢的问题:
>可以看到，程序停在了下面这一行。
[DEBUG] Searching for remote catalog: http://repo1.maven.org/maven2/archetype-catalog.xml
去查询这个文件的时候网络比较差或者其他原因，导致挂在那里。
###解决方法很简单
>1. 把上述地址复制到浏览器中，下载这个文件到本地。
2. 把文件archetype-catalog.xml复制到目录.m2\repository\org\apache\maven\archetype\archetype-catalog\2.2下面。
3. 在上述命令后增加参数-DarchetypeCatalog=local，变成读取本地文件即可。


##2、查看jar的编译jdk版本号
####1、解压jar,在其中一个class文件下启动命令行输入
>在jar包中，用winrar解压一个类文件，然后在命令行下面输入
javap -verbose classname
会输出一些信息，大致如下:

>Compiled from "HtmlCrawer.java"
public class org.eagleeye.html.HtmlCrawer extends java.lang.Object
SourceFile: "HtmlCrawer.java"
minor version: 0
major version: 50
Constant pool:
const #1 = class #2; //  org/eagleeye/html/HtmlCrawer
const #2 = Asciz org/eagleeye/html/HtmlCrawer;
const #3 = class #4; //  java.lang/Object
const #4 = Asciz java.lang/Object;
const #5 = Asciz client;
....
后面省略了，可以看到前面有两行:
>minor version: 0
major version: 50
表示了类文件的版本

####2、参考jdk的版本号对照表


##3、关于jsp在火狐下的显示不正常
>Html显示正常但是相同的jsp在chrome下正常而在firefox下不正常的原因:
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">替换掉jsp里面默认生成的<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN">。替换之后 发现问题解决了

##3、安装jar到本地仓库
>执行本地cmd命令
mvn install:install-file -Dfile=D:\kaptcha-2.3.2.jar -DgroupId=com.google.code.kaptcha -DartifactId=kaptcha -Dversion=2.3.2 -Dpackaging=jar