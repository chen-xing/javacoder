## maven 详解-史上最全的maven教程

### 摘要

详解要少仓库,maven命令创建项目，maven发布代码到仓库，maven常用命令，maven命令参数，maven三方仓库，maven骨架图

### 关键字

maven命令，maven骨架图

### 1、常用maven仓库

+ http://mvnrepository.com/
+ http://search.maven.org/
+ https://repository.sonatype.org/



### 2、生成项目

#### 2.1、创建Maven的普通Java项目

```
mvn archetype:create
    -DgroupId=packageName
    -DartifactId=projectName
```

#### 2.2、创建Maven的Web项目

```
mvn archetype:create
    -DgroupId=packageName
    -DartifactId=webappName
    -DarchetypeArtifactId=maven-archetype-webapp
```

#### 2.3、反向生成 maven 项目的骨架

```
mvn archetype:generate
```



### 3、发布代码到远程仓库

```
mvn install:install-file -DgroupId=com -DartifactId=client -Dversion=0.1.0 -Dpackaging=jar -Dfile=d:\client-0.1.0.jar
-DdownloadSources=true
-DdownloadJavadocs=true
```



### 4、常用命令

| 命令                   | 描述                                                         |
| ---------------------- | ------------------------------------------------------------ |
| mvn –version           | 显示版本信息                                                 |
| mvn clean              | 清理项目生产的临时文件,一般是模块下的target目录              |
| mvn compile            | 编译源代码，一般编译模块下的src/main/java目录                |
| mvn package            | 项目打包工具,会在模块下的target目录生成jar或war等文件        |
| mvn test               | 测试命令,或执行src/test/java/下junit的测试用例.              |
| mvn install            | 将打包的jar/war文件复制到你的本地仓库中,供其他模块使用       |
| mvn deploy             | 将打包的文件发布到远程参考,提供其他人员进行下载依赖          |
| mvn site               | 生成项目相关信息的网站                                       |
| mvn eclipse:eclipse    | 将项目转化为Eclipse项目                                      |
| mvn dependency:tree    | 打印出项目的整个依赖树                                       |
| mvn archetype:generate | 创建Maven的普通java项目                                      |
| mvn tomcat:run         | 在tomcat容器中运行web应用                                    |
| mvn jetty:run          | 调用 Jetty 插件的 Run 目标在 Jetty Servlet 容器中启动 web 应用 |

> 注意：运行maven命令的时候，首先需要定位到maven项目的目录，也就是项目的pom.xml文件所在的目录。否则，必以通过参数来指定项目的目录。



### 5、命令参数
| 命令 | 描述                                                         |
| ---- | ------------------------------------------------------------ |
| -D   | 以`-D`开头，将`maven.test.skip`的值设为`true`,就是告诉maven打包的时候跳过单元测试 |
| -P   | 使用指定的Profile配置                                        |
| -e   | 显示maven运行出错的信息                                      |
| -o   | 离线执行命令,即不去远程仓库更新包                            |
| -X   | 显示maven允许的debug信息                                     |
| -U   | 强制去远程更新snapshot的插件或依赖，默认每天只更新一次       |


### 6、三方仓库

```
<mirror>
    <id>nexus-aliyun</id>
    <name>Nexus aliyun</name>
    <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
    <mirrorOf>central</mirrorOf>
</mirror>

<mirror>
    <id>repo2</id>
    <name>Mirror from Maven Repo2</name>
    <url>http://repo2.maven.org/maven2/</url>
    <mirrorOf>central</mirrorOf>
</mirror>

<mirror>
    <id>ui</id>
    <name>Mirror from UK</name>
    <url>http://uk.maven.org/maven2/</url>
    <mirrorOf>central</mirrorOf>
</mirror>

<mirror>
    <id>jboss-public-repository-group</id>
    <mirrorOf>central</mirrorOf>
    <name>JBoss Public Repository Group</name>
    <url>http://repository.jboss.org/nexus/content/groups/public</url>
</mirror>
```



### 7、maven骨架图
|  archetype    |  说明    |
| ---------------------- | ---------------------- |
|appfuse-basic-jsf |(创建一个基于Hibernate，Spring和JSF的Web应用程序的原型)|
|appfuse-basic-spring |(创建一个基于Hibernate，Spring和Spring MVC的Web应用程序的原型)|
|appfuse-basic-struts |(创建一个基于Hibernate，Spring和Struts 2的Web应用程序的原型)|
|appfuse-basic-tapestry |(创建一个基于Hibernate, Spring 和 Tapestry 4的Web应用程序的原型)|
|appfuse-core |(创建一个基于 Hibernate and Spring 和 XFire的jar应用程序的原型)|
|appfuse-modular-jsf |(创建一个基于 Hibernate，Spring和JSF的模块化应用原型)|
|appfuse-modular-spring |(创建一个基于 Hibernate, Spring 和 Spring MVC 的模块化应用原型)|
|appfuse-modular-struts |(创建一个基于 Hibernate, Spring 和 Struts 2 的模块化应用原型)|
|appfuse-modular-tapestry |(创建一个基于 Hibernate, Spring 和 Tapestry 4 的模块化应用原型)| |
|maven-archetype-j2ee-simple |(一个简单的J2EE的Java应用程序)|
|maven-archetype-marmalade-mojo |(一个Maven的 插件开发项目 using marmalade)|
|maven-archetype-mojo |(一个Maven的Java插件开发项目)|
|maven-archetype-portlet |(一个简单的portlet应用程序)|
|maven-archetype-profiles |()|
|maven-archetype-quickstart |()|
|maven-archetype-site-simple |(简单的网站生成项目)|
|maven-archetype-site |(更复杂的网站项目)|
|maven-archetype-webapp |(一个简单的Java Web应用程序)|
|jini-service-archetype |(Archetype for Jini service project creation)|
|softeu-archetype-seam |(JSF+Facelets+Seam Archetype)|
|softeu-archetype-seam-simple |(JSF+Facelets+Seam (无残留) 原型)|
|softeu-archetype-jsf |(JSF+Facelets 原型)|
|jpa-maven-archetype |(JPA 应用程序)|
|spring-osgi-bundle-archetype |(Spring-OSGi 原型)|
|confluence-plugin-archetype |(Atlassian 聚合插件原型)|
|jira-plugin-archetype |(Atlassian JIRA 插件原型)|
|maven-archetype-har |(Hibernate 存档)|
|maven-archetype-sar |(JBoss 服务存档)|
|wicket-archetype-quickstart |(一个简单的Apache Wicket的项目)|
|scala-archetype-simple |(一个简单的scala的项目)|
|lift-archetype-blank |(一个 blank/empty liftweb 项目)|
|lift-archetype-basic |(基本（liftweb）项目)|
|cocoon-22-archetype-block-plain |([http://cocoapacorg2/maven-plugins/])|
|cocoon-22-archetype-block |([http://cocoapacorg2/maven-plugins/])|
|cocoon-22-archetype-webapp |([http://cocoapacorg2/maven-plugins/])|
|myfaces-archetype-helloworld |(使用MyFaces的一个简单的原型)|
|myfaces-archetype-helloworld-facelets |(一个使用MyFaces和Facelets的简单原型)|
|myfaces-archetype-trinidad |(一个使用MyFaces和Trinidad的简单原型)|
|myfaces-archetype-jsfcomponents |(一种使用MyFaces创建定制JSF组件的简单的原型)|
|gmaven-archetype-basic |(Groovy的基本原型)|
|gmaven-archetype-mojo |(Groovy mojo 原型)|
