### webmagic爬虫

### 摘要

webmagic爬虫入门实例，简单的实例，带你入门爬虫

### 关键字

webmagic，webmagic demo

### 1、引入pom

```
<dependency>
	<groupId>us.codecraft</groupId>
	<artifactId>webmagic-core</artifactId>
	<version>0.7.3</version>
	<exclusions>
		<exclusion>
			<groupId>org.slf4j</groupId>
			<artifactId>slf4j-log4j12</artifactId>
		</exclusion>
	</exclusions>
</dependency>
<dependency>
	<groupId>us.codecraft</groupId>
	<artifactId>webmagic-extension</artifactId>
	<version>0.7.3</version>
</dependency>
```

### 2、编写接收的bean

```
package tech.chenxing.webmagic;

import lombok.Data;
import us.codecraft.webmagic.Site;
import us.codecraft.webmagic.model.ConsolePageModelPipeline;
import us.codecraft.webmagic.model.OOSpider;
import us.codecraft.webmagic.model.annotation.ExtractBy;
import us.codecraft.webmagic.model.annotation.TargetUrl;

//@TargetUrl("https://www.94rg.com/sitemap.html")
@ExtractBy(value = "//*[@id=\"myTable\"]/li/div[1]/a", multi = true)
@Data
public class RgBlogUrlBean {
    @ExtractBy("//a/@href")
    private String url;

    @ExtractBy("//a/@title")
    private String title;
}
```

+ 类上的注解ExtractBy的属性 multi 说明返回的结果是一个列表
+ 属性上的ExtractBy的内容是可以利用chrome浏览器的右键复制xpath可以获取到
+ 多级属性用**/**分割，属性值获取采用 @属性值的方式获取，比如 @href

### 3、编写处理结果器

```
    public class RgPageModelPipeline implements PageModelPipeline<RgBlogUrlBean> {

    @Override
    public void process(RgBlogUrlBean rgBlogUrlBean, Task task) {
        System.out.println(rgBlogUrlBean.getUrl());
//        log.info("{}",rgBlogUrlBean.getUrl());
    }
}
```

+ 泛型值传入你想反序列化的bean,实现的接口内部编写你的处理逻辑，可以打印日志，保存到数据库，基于结果调用三方接口



### 4、编写启动的函数

```
public static void main(String[] args) {
    OOSpider.create(Site.me(), new RgPageModelPipeline(), RgBlogUrlBean.class)
    .addUrl("https://www.94rg.com/sitemap.html")
    .thread(1)
    .run();
}
```

可以说是非常的简便，可以用来抓取一些自己想要的数据。