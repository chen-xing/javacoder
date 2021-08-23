### java 反射

### 摘要

java的反射（reflection）机制是指在程序的运行状态中，可以构造任意一个类的对象，可以了解任意一个对象所属的类，可以了解任意一个类的成员变量和方法，可以调用任意一个对象的属性和方法。这种动态获取程序信息以及动态调用对象的功能称为Java语言的反射机制。反射被视为动态语言的关键。



### 关键

反射，reflection，invoke



### 1、什么是反射

​	Java的反射（reflection）机制是指在程序的运行状态中，可以构造任意一个类的对象，可以了解任意一个对象所属的类，可以了解任意一个类的成员变量和方法，可以调用任意一个对象的属性和方法。这种动态获取程序信息以及动态调用对象的功能称为Java语言的反射机制。反射被视为动态语言的关键。



### 2、反射的应用场景

+ 运行时动态根据配置动态创建对象，实现解耦（spring中比较多）
+ 动态代理
+ 动态修复一些三方jar的实现。（javassist asm）
+ jdbc数据库连接

### 3、反射中的关键对象

| 类名        | 说明         |
| ----------- | ------------ |
| Class       | 类的实例     |
| Field       | 类的成员变量 |
| Method      | 类的方法     |
| Constructor | 类的构造方法 |



### 4、class类

#### 4.1、获得类相关的方法

| 方法                         | 说明                                                         |
| ---------------------------- | ------------------------------------------------------------ |
| `asSubclass(Class<U> clazz)` | 把传递的类的对象转换成代表其子类的对象                       |
| `cast(Object obj)`           | 把对象转换成代表类或是接口的对象                             |
| `getClassLoader()`           | 获得类的加载器                                               |
| `getClasses()`               | 返回一个数组，数组中包含该类中所有公共（public）类和接口类的对象 |
| `getDeclaredClasses()`       | 返回一个数组，数组中包含该类中所有类和接口类的对象           |
| `forName(String className)`  | 根据类名返回类的对象                                         |
| `getName()`                  | 获得类的完整路径名字                                         |
| `newInstance()`              | 创建类的实例                                                 |
| `getPackage()`               | 获得类的包                                                   |
| `getSimpleName()`            | 获得类的名字                                                 |
| `getSuperclass()`            | 获得当前类继承的父类的名字                                   |
| `getInterfaces()`            | 获得当前类实现的类或是接口                                   |

#### 4.2、获取类属性的方法

| 方法                            | 说明                   |
| ------------------------------- | ---------------------- |
| `getField(String name)`         | 获得某个公有的属性对象 |
| `getFields()`                   | 获得所有公有的属性对象 |
| `getDeclaredField(String name)` | 获得某个属性对象       |
| `getDeclaredFields()`           | 获得所有属性对象       |

#### 4.3、获得类中注解相关的方法

| 方法                                              | 说明                                   |
| ------------------------------------------------- | -------------------------------------- |
| `getAnnotation(Class<A> annotationClass)`         | 返回该类中与参数类型匹配的公有注解对象 |
| `getAnnotations()`                                | 返回该类所有的公有注解对象             |
| `getDeclaredAnnotation(Class<A> annotationClass)` | 返回该类中与参数类型匹配的所有注解对象 |
| `getDeclaredAnnotations()`                        | 返回该类所有的注解对象                 |

#### 4.4、获得类中构造器相关的方法

| 方法                                                 | 说明                                   |
| ---------------------------------------------------- | -------------------------------------- |
| `getConstructor(Class...<?> parameterTypes)`         | 获得该类中与参数类型匹配的公有构造方法 |
| `getConstructors()`                                  | 获得该类的所有公有构造方法             |
| `getDeclaredConstructor(Class...<?> parameterTypes)` | 获得该类中与参数类型匹配的构造方法     |
| `getDeclaredConstructors()`                          | 获得该类所有构造方法                   |

#### 4.5、获得类中方法相关的方法

| 方法                                                         | 说明                   |
| ------------------------------------------------------------ | ---------------------- |
| `getMethod(String name, Class...<?> parameterTypes)`         | 获得该类某个公有的方法 |
| `getMethods()`                                               | 获得该类所有公有的方法 |
| `getDeclaredMethod(String name, Class...<?> parameterTypes)` | 获得该类某个方法       |
| `getDeclaredMethods()`                                       | 获得该类所有方法       |

#### 4. 6、其它重要方法

| 方法                                                         | 说明                             |
| ------------------------------------------------------------ | -------------------------------- |
| `isAnnotation()`                                             | 如果是注解类型则返回true         |
| `isAnnotationPresent(Class<? extends Annotation> annotationClass)` | 如果是指定类型注解类型则返回true |
| `isAnonymousClass()`                                         | 如果是匿名类则返回true           |
| `isArray()`                                                  | 如果是一个数组类则返回true       |
| `isEnum()`                                                   | 如果是枚举类则返回true           |
| `isInstance(Object obj)`                                     | 如果obj是该类的实例则返回true    |
| `isInterface()`                                              | 如果是接口类则返回true           |
| `isLocalClass()`                                             | 如果是局部类则返回true           |



### 5、Field类

成员变量（属性）操作相关的类

| 方法                          | 说明                    |
| ----------------------------- | ----------------------- |
| equals(Object obj)            | 属性与obj相等则返回true |
| get(Object obj)               | 获得obj中对应的属性值   |
| set(Object obj, Object value) | 设置obj中对应属性值     |



### 6、Method类

方法操作相关的类

| 方法                                 | 说明                                     |
| ------------------------------------ | ---------------------------------------- |
| `invoke(Object obj, Object... args)` | 传递object对象及参数调用该对象对应的方法 |

### 7、Constructor类

| 方法                              | 说明                       |
| --------------------------------- | -------------------------- |
| `newInstance(Object... initargs)` | 根据传递的参数创建类的对象 |



### 8、代码示例

#### 8.1、示例操作类

```
package com.whw.domain;

public class Customer {
    private Integer num;
    private String name;
    private String sex;
    private Integer age;
    private String addr;
    private Integer tel;

    public Customer(){

    }

    public Customer(Integer num, String name) {
        this.num = num;
        this.name = name;
    }

    public Customer(Integer num, String name, String sex, Integer age) {
        this.num = num;
        this.name = name;
        this.sex = sex;
        this.age = age;
    }

    public Customer(Integer num, String name, String sex, Integer age, String addr, Integer tel) {
        this.num = num;
        this.name = name;
        this.sex = sex;
        this.age = age;
        this.addr = addr;
        this.tel = tel;
    }

    public Integer getNum() {
        return num;
    }

    public void setNum(Integer num) {
        this.num = num;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getSex() {
        return sex;
    }

    public void setSex(String sex) {
        this.sex = sex;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    public String getAddr() {
        return addr;
    }

    public void setAddr(String addr) {
        this.addr = addr;
    }

    public Integer getTel() {
        return tel;
    }

    public void setTel(Integer tel) {
        this.tel = tel;
    }
}
```



#### 8.2、class类的创建

Class类的创建方式常用的有三种：

```
private static void operationClass() {
    //方式一：通过全类名的形式
    Class c1 = Customer.class;
    
    //方式一：通过全类名的形式
    Class c2= null;
    try {
        c2 = Class.forName("com.whw.domain.Customer");
    } catch (ClassNotFoundException e) {
        e.printStackTrace();
    }
    
	//方式三：对象.getClass()
    Class c2=new Customer().getClass();
}
```



#### 8.3、类相关操作方法

```
private static void operationClass2() {
        Class c1 = JButton.class;

        //获取类的完整路径名字
        String name = c1.getName();
        System.out.println(name);//javax.swing.JButton

        //获得类的包
        //Package aPackage = c1.getPackage();
        //System.out.println(aPackage);//package javax.swing, Java Platform API Specification, version 1.8
        //获取类的包名
        //System.out.println(aPackage.getName());//javax.swing

        //获取类的名字
        //String simpleName = c1.getSimpleName();
        //System.out.println(simpleName);//JButton

        //获得当前类继承的父类的名字
        //Class superclass = c1.getSuperclass();
        //System.out.println(superclass);//class javax.swing.AbstractButton
    }
3、获取类的所有构造方法
private static void operationClass3() 
    throws IllegalAccessException, InvocationTargetException, InstantiationException {
        /*Class c1 = Customer.class;
        Constructor[] constructors = c1.getConstructors();
        for (Constructor constructor : constructors) {
            int count = constructor.getParameterCount();
            System.out.println("构造函数：" + constructor.getName() + "，参数个数：" + count);
        }*/
    	//构造函数：com.whw.domain.Customer，参数个数：6
	//构造函数：com.whw.domain.Customer，参数个数：4
	//构造函数：com.whw.domain.Customer，参数个数：2
	//构造函数：com.whw.domain.Customer，参数个数：0
    

        //直接创建对象
        //Class c1 = Customer.class;
        //Customer customer = (Customer)c1.newInstance();
        //customer.setName("王伟");
        //System.out.println(customer.getName());//王伟


        //通过构造器构造对象
        //Class c1 = Customer.class;
        //Customer customer = (Customer)c1.getConstructors()[2].newInstance(1,"王伟");
        //System.out.println(customer.getName());//王伟
    }
```

#### 8.4、操作Filed

```
private static void operationClass4() 
    throws NoSuchFieldException, IllegalAccessException, InstantiationException {
        Class c1 = Customer.class;
        //获取所有公有public属性
        /*Field[] fields = c1.getFields();
        for (Field field:fields){
            System.out.println(field.getName());
        }*/

        //获取所有属性
        /*Field[] fields = c1.getDeclaredFields();
        for (Field field:fields){
            System.out.println("字段名："+field.getName()+",访问修饰符："+ Modifier.toString(field.getModifiers()));
        }*/
        //num
        //name
        //sex
        //age
        //addr
        //tel

        //获取指定属性,并设置值
        Customer customer= (Customer) c1.newInstance();
        Field field=c1.getDeclaredField("name");//获取姓名属性
        field.setAccessible(true);//设置可操作私有字段属性
        field.set(customer,"王伟");
        System.out.println(customer.getName());
}
```

#### 8.5、操作方法

```
private static void operationClass5() {
        Class c1 = Customer.class;
        Method[] methods = c1.getMethods();
        for (Method method : methods) {
            System.out.println(
                "方法名：" + method.getName() + 
                ",参数个数：" + method.getParameterCount()+
                "，返回值类型："+method.getGenericReturnType());
            Class<?>[] parameterTypes = method.getParameterTypes();
            for (Class parameterType : parameterTypes) {
                System.out.println(parameterType.getName());//获取参数类型
         }
     }
}
```



### 9、反射的优劣势

+ 优势：运行期类型的判断，动态类加载：提高代码灵活度
+ 劣势：性能瓶颈：反射相当于一系列解释操作，通知 JVM 要做的事情，性能比直接的java代码要慢很多

