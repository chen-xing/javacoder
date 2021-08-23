### postman Pre-request Script基础

### 1、背景

百度提供了自动推送的功能，但是每日推送的是有数量限制，目前我的限制在10个。之前是按照百度通过的规则在postman内部写死了要推送的url,这么做的缺陷是每次都得去手动修改body的内容。当然写个代码配合定时任务那完全不是事，只能主要说明的是如何利用postman去实现这个需求。毕竟postman的应用范围还是很广的。

### 2、需求描述

因为我的博文链接还很有规律的，比如：https://www.94rg.com/article/1800

所以我的需求可以归结为：

+ 设置一个起始值，对应文章的id,并且需要长期有效
+ 可以动态去设置body的内容

### 3、postman Pre-request的基础

| 语法 | 说明 |
| ---- | ---- |
|  pm.globals.set("variable_key", "variable_value");    |  设置全局变量    |
| pm.globals.get("variable_key"); | 获取全局变量 |
| pm.environment.set("variable_key", "variable_value"); | 设置环境变量 |
| pm.environment.get("variable_key"); | 获取环境变量 |
| pm.globals.unset("variable_key"); | 清理全局变量 |
| pm.globals.unset("variable_key"); | 清理全局变量 |
| pm.environment.unset("variable_key"); | 清理环境变量 |
| pm.variables.get("variable_key"); | 获取一个变量 |
| pm.sendRequest("https://postman-echo.com/get", function (err, response) {<br/>    console.log(response.json());<br/>}); | 发送一个request请求 |

### 4、重点说明

postman Pre-request的语法主要注意2个点

+ 语法基础参照3

+ 其他的语法其实就是普通**的javascript的语法**，常见的for循环的都是可以用js来实现的

  



### 5、代码成果

```
var articleId=pm.globals.get("articleId");
var urlList='';
for(var i=0;i<10;i++){
    var temp='https://www.94rg.com/article/'+(articleId-i);
    urlList=urlList+temp+"\n";
}
pm.globals.set("articleId",(articleId-10));
console.log(urlList);
pm.globals.set("urlList",urlList);
```

然后在body里面直接使用变量 **{{urlList}}**



