### 如何更新三方docker镜像的内容

### 摘要

对服务的源码进行了升级改造，但是对应的服务依赖的环境比较复杂，基于docker我们如何实现快速更新部署？



### 关键字

如何进入docker容器，docker拷贝文件到容器

### 1、背景

+ 对服务的源码进行了升级改造
+ 对应服务提供了docker镜像
+ 对应的服务依赖的环境比较复杂

### 2、有效的解决思路

+ 修改源码
+ 重新编译
+ 复制编译后的文件到docker容器中



### 3、如何操作

#### 3.1、启动容器

```
docker ps -a  
docker start xxx
```

#### 3.2、进入容器

```
docker exec -it 35802988f7fd sh
其中下面2个报错
docker exec -it 35802988f7fd /bin/bash 
docker exec -it 35802988f7fd bash
```

#### 3.3、查找pip的安装目录

```
pip --version
查找python pip安装的目录
```

#### 3.4、复制宿主机到容器

```
docker cp /usr/local/golden/tools/python3/lib/python3.6/site-packages/xmind2testlink 35802988f7fd:/usr/lib/python3.8/site-packages
```

#### 3.5、重启容器

```
docker restart xxxx
```



