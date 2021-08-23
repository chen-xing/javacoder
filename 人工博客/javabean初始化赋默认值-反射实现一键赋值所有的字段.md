### javabean初始化赋默认值-反射实现一键赋值所有的字段



### 摘要

对接的同学提供一个jar过来，然后里面的bean是4级嵌套，然后也没有个文档。看着这么多的属性心里有点蒙。看惯了swaggger的文档格式，心里在想，有没有一个方式可以实现这个需求。网上找了没有，然后自己动手了。

### 关键字

反射一键初始化bean、序列化保留多级null



### 1、问题背景

​	对接的同学提供一个jar过来，然后里面的bean是4级嵌套，也没有个文档。看着这么多的属性心里有点蒙。看惯了swaggger的文档格式，心里在想，有没有一个方式可以实现这个需求



### 2、问题本质

+ 把一个bean的结构用json的方式展示出来



### 3、难点

+ bean的结构是多级的
+ bean中包含自定义类
+ bean中包含静态类、内部类



### 4、可能的实现方式

#### 4.1、利用三方序列化工具，比如Gson实现

```
XXXTaskAddInput  xxxTaskAddInput=new XXXTaskAddInput();
Gson gson=new GsonBuilder().serializeNulls().create();
System.out.println(gson.toJson(xxxTaskAddInput));
```

结果

```
{
    "groupId": null, 
    "groupName": null, 
    "tasks": null
}
```

从结果明显可以看出，只是保留了一级的null属性。所以说Gson的serializeNulls这个功能是不满足我的这个需求的。



#### 4.2、专注到初始化话

不难看出问题的本质不是序列化，而在于bean的初始化。因为一级属性都是null。二级属性自然是无法序列化出来的。所以我们的核心是实现变量bean，给所有层级的属性赋值对应类型的默认值就好了。

简单的查了下百度，没有直接可用的，然后就自己动手了。反射是解决这个问题的关键。



### 5、可复用的工具类

```
public static void setFeidValueNotNull(Object obj) throws Exception {
        for (Field field : obj.getClass().getDeclaredFields()) {
            field.setAccessible(true);
            if (field.get(obj) == null || field.get(obj).toString().equals("[]")) {
                if (field.getGenericType().toString().equals("class java.lang.String")) {
                    field.set(obj, "");
                } else if (field.getGenericType().toString().equals("class java.lang.Integer")) {
                    field.set(obj, 0);
                } else if (field.getGenericType().toString().equals("class java.lang.Double")) {
                    field.set(obj, 0.0);
                } else if (field.getGenericType().toString().equals("class java.lang.Long")) {
                    field.set(obj, 0L);
                } else {
                    Type type = field.getGenericType();
                    if (List.class.isAssignableFrom(field.getType())) {
                        List arraylist = new ArrayList();
                        // 这样判断type 是不是参数化类型。 如Collection<String>就是一个参数化类型。
                        if (type instanceof ParameterizedType) {
                            // 获取类型的类型参数类型。  你可以去查看jdk帮助文档对ParameterizedType的解释。
                            Class clazz =
                                    (Class) ((ParameterizedType) type).getActualTypeArguments()[0];
                            Object subObject = clazz.newInstance();
                            setFeidValueNotNull(subObject);
                            arraylist.add(subObject);
                        }
                        field.set(obj, arraylist);
                    } else {
                        Class clazz = Class.forName(field.getGenericType().getTypeName());
                        Object subObject = clazz.newInstance();
                        setFeidValueNotNull(subObject);
                        field.set(obj, subObject);
                    }
                }
            }
        }
    }
```

如何使用

```
XXXTaskAddInput  xxxTaskAddInput=new XXXTaskAddInput();
setFeidValueNotNull(xxxTaskAddInput);
System.out.println(new Gson().toJson(xxxTaskAddInput));
```

效果

```
{
    "groupId": "", 
    "groupName": "", 
    "tasks": [
        {
            "bizId": "", 
            "bizInfo": {
                "receiver": "", 
                "extendInfo": "", 
                "organizationId": "", 
                "exportInfos": [
                    {
                        "columnIndex": 0, 
                        "rowspan": 1, 
                        "columnName": "", 
                        "columnValue": ""
                    }
                ], 
                "result": ""
            }, 
            "name": "", 
            "type": 0, 
            "total": 0, 
            "accountOid": "", 
            "accountGid": "", 
            "receiver": ""
        }
    ]
}
```

是不是没有想象中复杂，补充一句，swagger中是借助于注解实现的。

### 6、总结

发现问题，解决问题，总结问题。一步一个脚印的往前走。回头看，其实所谓的困难也没那么难。