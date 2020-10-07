### hbase的基本操作命令

#### 1、命令列表

|命名|描述|语法|
| ---- | ---- | ---- |
|help ‘命名名’|查看命令的使用描述|help ‘命令名’|
|whoami|我是谁|whoami|
|version|返回hbase版本信息|version|
|status|返回hbase集群的状态信息|status|
|table_help|查看如何操作表|table_help|
|create|创建表|create ‘表名’, ‘列族名1’, ‘列族名2’, ‘列族名N’|
|alter|修改列族|添加一个列族：alter ‘表名’, ‘列族名’   删除列族：alter ‘表名’, {NAME=&gt; ‘列族名’, METHOD=&gt; ‘delete’}|
|describe|显示表相关的详细信息|describe ‘表名’|
|list|列出hbase中存在的所有表|list|
|exists|测试表是否存在|exists ‘表名’|
|put|添加或修改的表的值|put ‘表名’, ‘行键’, ‘列族名’, ‘列值’ put ‘表名’, ‘行键’, ‘列族名:列名’, ‘列值’|
|scan|通过对表的扫描来获取对用的值|scan ‘表名’  扫描某个列族： scan ‘表名’, {COLUMN=&gt;‘列族名’} 扫描某个列族的某个列： scan ‘表名’, {COLUMN=&gt;‘列族名:列名’}  查询同一个列族的多个列： scan ‘表名’, {COLUMNS =&gt; [ ‘列族名1:列名1’, ‘列族名1:列名2’, …]}|
|get|获取行或单元（cell）的值|get ‘表名’, ‘行键’  get ‘表名’, ‘行键’, ‘列族名’|
|count|统计表中行的数量|count ‘表名’|
|incr|增加指定表行或列的值|incr ‘表名’, ‘行键’, ‘列族:列名’, 步长值|
|get_counter|获取计数器|get_counter ‘表名’, ‘行键’, ‘列族:列名’|
|delete|删除指定对象的值（可以为表，行，列对应的值，另外也可以指定时间戳的值）|删除列族的某个列： delete ‘表名’, ‘行键’, ‘列族名:列名’|
|deleteall|删除指定行的所有元素值|deleteall ‘表名’, ‘行键’|
|truncate|重新创建指定表|truncate ‘表名’|
|enable|使表有效|enable ‘表名’|
|is_enabled|是否启用|is_enabled ‘表名’|
|disable|使表无效|disable ‘表名’|
|is_disabled|是否无效|is_disabled ‘表名’|
|drop|删除表|drop的表必须是disable的 disable ‘表名’ drop ‘表名’|
|shutdown|关闭hbase集群（与exit不同）||
|tools|列出hbase所支持的工具||
|exit|退出hbase shell||
|list_namespace|列举命名空间||
|describe_namespace|获取命名空间描述||
|list_namespace_tables|查看命名空间下的所有表||
|create_namespace|创建命名空间||
|show_filters|显示hbase所支持的所有过滤器||

#### 2、应用举例

##### 2.1、TIMESTAMP 指定时间戳

```
scan 't1', {COLUMNS => 'c2', TIMESTAMP=> 1552819392398}
```

##### 2.2、TIMERANGE表示的是”>=开始时间 and <结束时间“

```
# 语法
scan '表名',{TIMERANGE=>[timestamp1, timestamp2]}

# 示例
scan 'tbl_user',{TIMERANGE=>[1551938004321, 1551938036450]}
```

##### 2.3、STARTROW

ROWKEY起始行。会先根据这个key定位到region，再向后扫描

```
# 语法
scan '表名', { STARTROW => '行键名'}

# 示例
scan 'tbl_user', { STARTROW => 'vbirdbest'}
```

##### 2.4、STOPROW ：截止到STOPROW行，STOPROW行之前的数据，不包括STOPROW这行数据

```
# 语法
scan '表名', { STOPROW => '行键名'}

# 示例
scan 'tbl_user', { STOPROW => 'vbirdbest'}
```

##### 2.5、LIMIT 返回的行数

```
# 语法
scan '表名', { LIMIT => 行数}

# 示例
scan 'tbl_user', { LIMIT => 2 }
```

##### 2.6、FILTER条件过滤器

过滤器之间可以使用AND、OR连接多个过滤器。

###### ValueFilter 值过滤器

```
# 语法：binary 等于某个值
scan '表名', FILTER=>"ValueFilter(=,'binary:列值')"
# 语法 substring:包含某个值
scan '表名', FILTER=>"ValueFilter(=,'substring:列值')"

# 示例
scan 'tbl_user', FILTER=>"ValueFilter(=, 'binary:26')"
scan 'tbl_user', FILTER=>"ValueFilter(=, 'substring:6')"
```

##### 2.7、ColumnPrefixFilter 列名前缀过滤器

```
# 语法 substring:包含某个值
scan '表名', FILTER=>"ColumnPrefixFilter('列名前缀')"

# 示例
scan 'tbl_user', FILTER=>"ColumnPrefixFilter('birth')"
# 通过括号、AND和OR的条件组合多个过滤器
scan 'tbl_user', FILTER=>"ColumnPrefixFilter('birth') AND ValueFilter(=,'substring:26')"
```

