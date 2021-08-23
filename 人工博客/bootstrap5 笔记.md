# bootstrap5建站笔记

## 1、引入依耐

```
<link href="https://cdn.jsdelivr.net/npm/bootstrap@5.0.2/dist/css/bootstrap.min.css" rel="stylesheet" integrity="sha384-EVSTQN3/azprG1Anm3QDgpJLIm9Nao0Yz1ztcQTwFspd3yD65VohhpuuCOmLASjC" crossorigin="anonymous">

<script src="https://cdn.jsdelivr.net/npm/bootstrap@5.0.2/dist/js/bootstrap.bundle.min.js" integrity="sha384-MrcW6ZMFYlzcLA8Nl+NtUVF0sA7MsXsP1UyJoMp4YLEuNSfAP+JcXn/tWtIaxVXM" crossorigin="anonymous"></script>
```

> http://www.fontawesome.com.cn/icons/arrow-circle-up/ 查询图标
>
> 为了增加图标大小相对于它们的容器, 使用 `fa-lg` (33% 递增), `fa-2x`, `fa-3x`, `fa-4x`, 或 `fa-5x` classes.

## 2、container

| Extra small <576px | Small ≥576px | Medium ≥768px | Large ≥992px | X-Large ≥1200px | XX-Large ≥1400px |        |
| ------------------ | ------------ | ------------- | ------------ | --------------- | ---------------- | ------ |
| `.container`       | 100%         | 540px         | 720px        | 960px           | 1140px           | 1320px |
| `.container-sm`    | 100%         | 540px         | 720px        | 960px           | 1140px           | 1320px |
| `.container-md`    | 100%         | 100%          | 720px        | 960px           | 1140px           | 1320px |
| `.container-lg`    | 100%         | 100%          | 100%         | 960px           | 1140px           | 1320px |
| `.container-xl`    | 100%         | 100%          | 100%         | 100%            | 1140px           | 1320px |
| `.container-xxl`   | 100%         | 100%          | 100%         | 100%            | 100%             | 1320px |
| `.container-fluid` | 100%         | 100%          | 100%         | 100%            | 100%             | 100%   |

## 3、返回顶部代码
```
 <a href="#top" class="el-backtop" style="right: 20px; bottom: 60px;">
 	<i class="fa fa-arrow-up fa" aria-hidden="true"></i>
 </a>
```

+ 在页面顶部定义一个锚点，id为top
+ style使用绝对定位，固定在右边右下角。
+ i标签 使用了fontawesonme的图标

## 4、统一的提示

> 调用toastr.js插件之前需要先引入jquery.js     toastr.js       toastr.css

```

<link href="https://cdn.bootcdn.net/ajax/libs/toastr.js/latest/css/toastr.css" rel="stylesheet">
<script src="https://cdn.bootcdn.net/ajax/libs/jquery/3.6.0/jquery.min.js"></script>
<script src="https://cdn.bootcdn.net/ajax/libs/toastr.js/latest/js/toastr.min.js"></script>
注意：toastr.js是基于jquery.js库，所以必须在toastr.js之前引入jquery.js
```

基本的语法

+  toastr.warning('你有新消息了！'); 
+  toastr.success('你有新消息了！');   
+  toastr.error('你有新消息了！');   
+  toastr.info('你有新消息了！');

## 5、js实现时间戳和日期相互转化

### 5.1、获取当前时间戳

```
var time = new Date().getTime(); //1603009495724,精确到毫秒
```

### 5.2、时间戳转日期

```
Date.prototype.Format = function (fmt) {
    var o = {
	"M+": this.getMonth() + 1, //月份
	"d+": this.getDate(), //日
	"h+": this.getHours(), //小时
	"m+": this.getMinutes(), //分
	"s+": this.getSeconds(), //秒
	"q+": Math.floor((this.getMonth() + 3) / 3), //季度
	"S": this.getMilliseconds() //毫秒
    };
    if (/(y+)/.test(fmt)) fmt = fmt.replace(RegExp.$1, (this.getFullYear() + "").substr(4 - RegExp.$1.length));
    for (var k in o)
	if (new RegExp("(" + k + ")").test(fmt)) fmt = fmt.replace(RegExp.$1, (RegExp.$1.length == 1) ? (o[k]) : (("00" + o[k]).substr(("" + o[k]).length)));
    return fmt;
}

/**
 * 反解析时间戳
 */
function analysisStamp() {
    var timestamp = $("#timestamptext").val();
    var newDate = new Date();
    newDate.setTime(timestamp);
    var commonTime = newDate.Format("yyyy-MM-dd hh:mm:ss.S");
    $("#timestamptext").val(commonTime);
}
```



## 6、js全部替换

```
uuid = uuid.replace(/-/g, "");
将字符串中的所以的 -替换掉
```

