# maven解决依赖冲突

### 1、使用场景
 工作中经常遇到最终使用的jar的版本并非自己pom文件中指定的版本。这个往往就是maven的依赖冲突问题<br />

### 2、maven的依赖原则

#### 2.1、最短路径优先原则
一个项目Demo依赖了两个jar包，其中A-B-C-X(1.0) ， A-D-X(2.0)。由于X(2.0)路径最短，所以项目使用的是X(2.0)。

#### 2.2、第一声明优先原则
 如果A-B-X(1.0) ，A-C-X(2.0) 这样的路径长度一样怎么办呢？这样的情况下，maven会根据pom文件声明的顺序加载，如果先声明了B，后声明了C，那就最后的依赖就会是X(1.0)。

#### 2.3、**覆写优先**
子pom内声明的优先于父pom中的依赖。

### 3、解决方案
   遇到冲突的时候第一步要找到maven加载的到时是什么版本的jar包，通过们`mvn dependency:tree`查看依赖树，通过maven的依赖原则来调整坐标在pom文件的申明顺序是最好的办法。
### 4、常用命令说明
> 主要用到的指令为  mvn dependency:tree ，在 pom.xml 所在的目录下执行**

> mvn dependency:tree  
> 会查看到**二级层级**的依赖关系 （并不能看到所有的依赖关系）


#### 查看完整的依赖关系，需要增加参数 -Dverbose
> 指令如下：

> **mvn dependency:tree  -Dverbose**

**通常我们只对产生冲突的部分感兴趣，这个时候我们就要对信息进行提取：需要增加以下参数

​    -Dincludes=*guava*注意：

​      includes 支持通配符的形式

> 执行指令：  mvn dependency:tree -Dverbose -Dincludes=*spring*:*spring*

