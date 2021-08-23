# idea代码模板如何设置？

## 1、模板分类

+ File and Code template 文件级别的模板
+ Live  template 方法或代码块级别的模板

## 2、File and Code template

![idea代码模板设置](https://cdn.jsdelivr.net/gh/chen-xing/figure_bed_02/cdn/20210728194743558.png)

```
/**
 * @description TODO
 * @date ${YEAR}-${MONTH}-${DAY} ${TIME}
 * @author chen xing
 */
```

其中${DATE}的样式是 2021/07/28,这里使用了${YEAR}-${MONTH}-${DAY}进行了日期的格式化，纯属个人强迫症。

按照上面的设置后，当新建.java文件的时候,默认会按照这个模版去生成对应的代码注释



## 3、Live  template

这个代码块的模板的功能比较强大，理论上可以玩出花来。常见的 

+ psvm (main方法)
+ sout (控制台输出)
+ fori(循环)
+ ...

但这次主要介绍的是方法级别的注释怎么弄

![idea方法级别的模版如何设置](https://cdn.jsdelivr.net/gh/chen-xing/figure_bed_02/cdn/20210728195657076.png)



简单的描述下几个步骤

+ 新建模板组和模板，操作对应第二步，结果对应第三步
+ 为新建的模板命名和添加备注,对应步骤中的第四步和第五步
+ 添加代码模板以及关联对应的文件类型。对应第六步
+ 设置触发的快捷键。对应第七步
+ 编辑变量



**template text**

```
**
* @author chen xing
* @description TODO
$param$
* @return $return$
* @date $date$ $time$
*/
```



**对应的变量**

| 变量   | 表达式             |
| ------ | ------------------ |
| time   | time()             |
| date   | date("yyyy-MM-dd") |
| param  | 见下方表达式       |
| return | methodReturnType() |

**param表达式**

```
groovyScript("  def result = '';  def param = \"${_1}\".replaceAll('[\\\\[|\\\\]|\\\\s]', '').split(',').toList();  for(int i = 0;i < param.size();i++)  {         result += '* @Param ' + param[i] + ((i < param.size() - 1) ? '\\n' : '');  }; return result; ",methodParameters()) 
```



**需要重点说明的问题**

+ **代码模板不能以/开头**，否则部分变量不生效(写在方法内部是可以的，但是没人愿意来回复制)
+ methodParameters() 默认是数组格式展示，可以用上面的表达式优化

