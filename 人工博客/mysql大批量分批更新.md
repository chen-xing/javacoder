### mysql大批量分批更新



### 关键字

mysql update语句与limit的结合使用

> 有些时候一条sql更新的记录行数是比较多的，比如常见的UPDATE USER SET pwd='xxx' WHERE pwd='12345';批量将默认密码重置为xxx。看似很平常的sql操作，那如果满足条件的用户数有百亿级别会怎么样。显然这么做是不可行的。

**我们是需要分批次限量进行更新的 。**



有时候有需要批量更新数据表中从多少行到多少行的某个字段的值
mysql的update语句只支持更新前多少行，不支持从某行到另一行，比如

```
UPDATE tb_name SET column_name='test' ORDER BY id ASC LIMIT 30;
```


更新前30行的某个字段内容，没什么问题。

```
UPDATE tb_name SET column_name='test' ORDER BY id ASC LIMIT 20,10;更新从20行到30行的某个字段的内容，这样会报错。
```

解决办法就是采用子查询的方式

```
UPDATE tb_name SET column_name='test' WHERE id in (SELECT id FROM (SELECT * FROM tb_name ORDER BY id ASC LIMIT 20,10) AS tt);
```


这样就能实现更新表中根据id升序排序的第20条到第30条数据的某个字段的内容
