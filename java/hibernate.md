##Hibernate使用总结
>1、参数化查询建议使用命名参数方式如select a.* from user a where a.name=:name
2、设置参数：单个对象的时候直接query.setParameter("name",value);当参数为集合的时候,query.setParameterList("name",value);
3、当参数的中包含like的时候，%%的拼接应用到参数中，sql中直接一个名称参数代替
4、当返回的结果包含一个唯一值的时候，query.addScalar("count"); query.uniqueResult().toString() 可以获得这个唯一值
5、当返回的结果映射成对象的时候query.addEntity(DocTemplate.class);
6、当返回的结果是Map的时候 query.setResultTransformer(Transformers.ALIAS_TO_ENTITY_MAP)
7、表中的默认字段被重置为null的时候，将jpa中的@Column(name = "modifydate",insertable =false,updatable = false)，这样插入和更新就不会包含这个字段