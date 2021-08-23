### freemarker语法

### 摘要

freemarker语法、freemarker入门、简单高效的模板引擎。



### 关键字

freemarker语法，freemarker入门



freemarker是一款简单高效的模版引擎。



### 1、基础语法

| 语法                | 说明       |
| ----------------------- | -------------- |
| ${expr}                 | 通用插值       |
| #{expr}或#{expr;format} | 数字格式化插值 |
| ${book.name?if_exists } | 用于判断如果存在,就输出这个值 |
| ${book.name?default(‘xxx’)} | 默认值xxx |
| ${book.name!"xxx"} | 默认值xxx |
| ${book.date?string('yyyy-MM-dd')} | 日期格式 |
| ${book?string.number} | 20 //三种不同的数字格式 |
| ${book?string.currency} | $20.00 |
| ${book?string.percent} | <#-- 20% --> |
| <#assign foo=ture /> | 声明变量,插入布尔值进行显示 |
| ${foo?string("yes","no")} | <#-- yes --> |
| <#assign foo=ture /> | 声明变量,插入布尔值进行显示 |
| ${book?string.percent} | <#-- 20% --> |
| <#assign foo=ture /> | 声明变量,插入布尔值进行显示 |
| ${book?string.percent} | <#-- 20% --> |
| <#assign foo=ture /> | 声明变量,插入布尔值进行显示 |
| ${book?string.percent} | <#-- 20% --> |
| <#assign foo=ture /> | 声明变量,插入布尔值进行显示 |

大小比较符号使用需要注意:(xml的原因),可以用于比较数字和日期
使用lt、lte、gt和gte来替代<、<=、>和>= 也可以使用括号<#if (x>y)>

内置函数: 调用区别于属性的访问,使用?代替.
常见的一些内置函数
对于字符串
html－对字符串进行HTML编码
cap_first－使字符串第一个字母大写
lower_case－将字符串转换成小写
trim－去掉字符串前后的空白字符

示例：${“freeMarker”?cap_first} 



### 2、集合

| 语法                | 说明       |
| ----------------------- | -------------- |
| <#assign l=0..100/> | 快速定义int区间的集合 |
| <#list student as stu>${stu}</#list> | 与jstl循环类似,也可以访问循环的状态item_index:当前变量的索引值item_has_next:是否存在下一个对象 其中item名称为as后的变量名,如stu |
| <#if student?size != 0></#if> | 判断=的时候,注意只要一个=符号,而不是== |

### 3、宏/模板

宏定义

```
<#macro greet>
<font size="+2">Hello Joe!</font>
</#macro>
```

模板

```
<@greet></@greet> //同xml可以简写成<@greet/>
```

宏的参数定义,类似js,在宏名后 带参数进行传递定义

```
<#macro greet person color>
${person}
</#macro>
```

调用带参数时,注意使用类似XML的属性格式进行传递,不需要关心顺序问题

```
<@greet person="Fred" color="black"/>
```



宏的循环变量,配合嵌套标签进行参数传递,

```
<#macro repeat count>
    <#list 1..count as x>
    <#nested x, x/2, x==count> //这里的三个参数,将会传递到嵌套内容中
    </#list>
</#macro>
```



函数定义:区别于宏对象,带返回值
<#function name param1 param2><#return val></#function>函数，有返回参数

stringA[M .. N] 取子字符串，类似substring(stringA, M, N)

<#include "/copyright_footer.html"> 导入其他页面元素
<#include filename options>
options包含两个属性
encoding=”GBK” 编码格式
parse=true 是否作为ftl语法解析,默认是true，false就是以文本方式引入.注意在ftl文件里布尔值都是直接赋值的如parse=true,而不是

parse=”true”

hash与list的定义
<#assign c= {"a":"orz","b":"czs"}>
${c.a}

List片段可以采用： products[10..19] or products[5..] 的格式进行定义,当只局限于数字
<#assign c= [1,2,3,4,5,6,6,7]>
<#list c[1..3] as v>
${v}
</#list>

对变量的缺省处理
product.color!"red"

用compress directive或者transform来处理输出。
<#compress>...</#compress>：消除空白行。
<@compress single_line=true>...</@compress>将输出压缩为一行。都需要包裹所需文档

freemarker可用"["代替"<".在模板的文件开头加上[#ftl].

注释部分
<#-- 注释部分 -->

数字输出的另外一种方式
\#{c.a;m0} 区别于${},这个例子是用于输出数字的格式化,保留小数的位数,详细如下

数字格式化插值可采用#{expr;format}形式来格式化数字,其中format可以是:
mX:小数部分最小X位
MX:小数部分最大X位

在定义字符串的时候,可以使用''或者"",对特殊字符,需要使用\进行转义

如果存在大量特殊字符,可以使用${r"..."}进行过滤
${r"${foo}"}
${r"C:\foo\bar"}

Map对象的key和value都是表达式,但是key必须是字符串
可以混合使用.和[""]访问
book.author["name"] //混合使用点语法和方括号语法

为了处理缺失变量,FreeMarker提供了两个运算符: 用于防止对象不存在而导致的异常
!:指定缺失变量的默认值
??:判断某个变量是否存在,返回boolean值

noparse指令指定FreeMarker不处理该指定里包含的内容,该指令的语法格式如下:
<#noparse>...</#noparse>

${firstName?html} 使用html对字符进行格式化处理,对于<等的过滤

escape , noescape指令,对body内的内容实用统一的表达式
看如下的代码:
<#escape x as x?html>
First name:${firstName}
Last name:${lastName}
Maiden name:${maidenName}
</#escape>
上面的代码等同于:
First name:${firstName?html}
Last name:${lastName?html}
Maiden name:${maidenName?html}

定义全局变量的方式
<#assign name1=value1 name2=value2 / > // 可以同时定义多个变量,也可以使用循环来给变量赋值
<#assign x>
<#list ["星期一", "星期二", "星期三", "星期四", "星期五", "星期六", "星期天"] as n>
${n}
</#list>
</#assign>
${x}

setting指令,用于动态设置freeMarker的运行环境:

该指令用于设置FreeMarker的运行环境,该指令的语法格式如下:<#setting name=value>,在这个格式中,name的取值范围包含如下几个:
locale:该选项指定该模板所用的国家/语言选项
number_format:指定格式化输出数字的格式
boolean_format:指定两个布尔值的语法格式,默认值是true,false
date_format,time_format,datetime_format:指定格式化输出日期的格式
time_zone:设置格式化输出日期时所使用的时区

<#return> 用于退出宏的运行

?html 用于将字符串中可能包含的html字符,进行过滤.

调用Java方法,需要使用实现TemplateMethodModel接口,但是好像会覆盖掉属性的访问



