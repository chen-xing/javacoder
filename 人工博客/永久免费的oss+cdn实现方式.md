## 永久免费的oss+cdn实现方式

### 摘要

cdn+oss+https组合起来，还永久免费，很多人听起来觉得我是在开玩笑，其实不然，还真的是有这么一个实现方式。



### 关键字

永久免费cdn,免费oss



### 1、前言

cdn+oss+https组合起来，还永久免费，很多人听起来觉得我是在开玩笑，其实不然，还真的是有这么一个实现方式。

注意额，我说的是永久免费，市面上仅有七牛云、又拍云提供少量的实名免费套餐，而且仅限于http。先来一个我基于这个理论实现的一个免费的图床给大家看下。体验地址：[https://www.chenzhuofan.top/figurebed](https://www.chenzhuofan.top/figurebed)

效果图如下：

![image-20201130133051128](https://cdn.jsdelivr.net/gh/chen-xing/figure_bed/images/20201130133058.png)



### 2、实现的原理

+ github实现资源文件的托管
+ jsdelivr实现github的文件的cdn加速。（国内访问github的速度的确是有点慢）

### 3、细节说明

#### 3.1、 GitHub配置

**1. 创建Repository**

鼠标移动到右上角，点击"New repository"按钮：

[![img](https://cdn.jsdelivr.net/gh/chen-xing/figure_bed/images/20201130134505.png)](https://cdn.jsdelivr.net/gh/dengfaheng/image01/20200628100544.png)

填写相关信息，创建一个存储图片的仓库：

[![img](https://cdn.jsdelivr.net/gh/chen-xing/figure_bed/images/20201130134514.png)](https://cdn.jsdelivr.net/gh/dengfaheng/image01/20200628100738.png)

**2. 配置token key**
生成一个Token用于操作GitHub repository。回到主页，点击"Settings"按钮：

[![img](https://cdn.jsdelivr.net/gh/chen-xing/figure_bed/images/20201130134524.png)](https://cdn.jsdelivr.net/gh/dengfaheng/image01/20200628101024.png)

进入页面后，点击"Developer settings"按钮

[![img](https://cdn.jsdelivr.net/gh/chen-xing/figure_bed/images/20201130134533.png)](https://cdn.jsdelivr.net/gh/dengfaheng/image01/20200628101303.png)

点击"Personal access tokens"按钮，然后点击Generate token：

[![img](https://cdn.jsdelivr.net/gh/chen-xing/figure_bed/images/20201130134540.png)](https://cdn.jsdelivr.net/gh/dengfaheng/image01/20200628101322.png)

填写描述，选择"repo"权限，然后拉到下面点击"Generate token"按钮

[![img](https://cdn.jsdelivr.net/gh/chen-xing/figure_bed/images/20201130134547.png)](https://cdn.jsdelivr.net/gh/dengfaheng/image01/20200628101404.png)
[![img](https://cdn.jsdelivr.net/gh/dengfaheng/image01/20200628101423.png)](https://cdn.jsdelivr.net/gh/dengfaheng/image01/20200628101423.png)

> 注意：创建成功后，会生成一串token，这串token之后不会再显示，所以第一次看到的时候，可以用个小本本保存起来哦，忘记了只有重新生成，每次都不一样。

详细教程参见：[https://www.cnblogs.com/chen-xing/p/14054218.html](https://www.cnblogs.com/chen-xing/p/14054218.html)

#### 3.2、 GitHub api的调用

核心的代码如下：

```
public static boolean create(String url, byte[] bytes, String token) {
        Map<String, String> bodyMap = Maps.newConcurrentMap();
        bodyMap.put("message", "图床提交");
        bodyMap.put("content", Base64.getEncoder().encodeToString(bytes));

        HttpResponse httpResponse =
                HttpRequest.put(url)
                        .header("Content-Type", "application/json")
                        .header("Authorization", MessageFormat.format("token {0}", token))
                        .body(new Gson().toJson(bodyMap)).timeout(4000)
                        .execute();
        boolean result=new Integer(201).equals(httpResponse.getStatus());
        if(!result){
            log.error("image upload github error:{}",httpResponse.body());
        }
        return result;
    }
```

这里只是涉及到上传文件，更多的github api v3的介绍参见：[https://www.cnblogs.com/chen-xing/p/14058096.html](https://www.cnblogs.com/chen-xing/p/14058096.html)

#### 3.3、jsdelivr 加速

j[sDelivr官网](https://www.jsdelivr.com/?docs=gh)提供了github、npm、wordpress这三个直观例子，有兴趣的道友可以去[官网](https://www.jsdelivr.com/?docs=gh)了解

一、访问github的用法

```
https://cdn.jsdelivr.net/gh/用户名称/仓库名称@版本号/目录
```

+ //加载资源(版本号不填的话，默认引用最新) [https://cdn.jsdelivr.net/gh/murenziwei/images/draw/01.png](https://cdn.jsdelivr.net/gh/murenziwei/images@master/draw/01.png)

+ //打开目录（后面的"/"是必要的，不然的话，打不开）https://cdn.jsdelivr.net/gh/murenziwei/images/

 

二、访问npm的用法

```
https:``//cdn.jsdelivr.net/npm/包名@版本号/目录
```

+ //加载资源 https://cdn.jsdelivr.net/npm/lw_firewords@1.0.3/index.js

+ //打开目录（后面的"/"是必要的，不然的话，打不开）[https://cdn.jsdelivr.net/](https://cdn.jsdelivr.net/gh/murenziwei/images/)[npm/lw_firewords/](https://cdn.jsdelivr.net/gh/murenziwei/images@master/draw/01.png)

 

三、访问wordpress的用法

+ // 加载任何插件从WordPress.org插件SVN repo https://cdn.jsdelivr.net/wp/plugins/project/tags/version/file
+ // 加载精确版本 https://cdn.jsdelivr.net/wp/plugins/wp-slimstat/tags/4.6.5/wp-slimstat.js
+ // 加载最新版本 // 你不应该在生产中使用这个 https://cdn.jsdelivr.net/wp/plugins/wp-slimstat/trunk/wp-slimstat.js
 + // 从WordPress.org的主题SVN repo加载任何主题 https://cdn.jsdelivr.net/wp/themes/project/version/file
+ // 加载精确版本  https://cdn.jsdelivr.net/wp/themes/twenty-eightteen/1.7/assets/js/html5.js
