### hibernate validator注解校验的使用

### 1、背景

对外提供的接口（包括restapi、jar）需要对入参进行一定规则的限制，比如只能是数字型、只能在一定的区间内、属性必须是枚举值中的一个。而这些简单的检验如果量很大的话，编写大量的堆积校验代码肯定是不太合适的。这时候注解校验就显得比较能够有效的解决这个局面。

### 2、引入pom

```
<dependency>
    <groupId>org.hibernate.validator</groupId>
    <artifactId>hibernate-validator</artifactId>
    <version>6.0.17.Final</version>
</dependency>
```



### 3、常见注解说明

#### 3.1、常用注解

| 注解               | 使用                                                         |
| ------------------ | ------------------------------------------------------------ |
| @NotNull           | 被注释的元素（任何元素）必须不为 null, 集合为空也是可以的。没啥实际意义 |
| @NotEmpty          | 用来校验字符串、集合、map、数组不能为null或空 （字符串传入空格也不可以）（集合需至少包含一个元素） |
| @NotBlank          | 只用来校验字符串不能为null，空格也是被允许的 。校验字符串推荐使用@NotEmpty |
| @Size(max=, min=)  | 指定的字符串、集合、map、数组长度必须在指定的max和min内 允许元素为null，字符串允许为空格 |
| @Length(min=,max=) | 只用来校验字符串，长度必须在指定的max和min内 允许元素为null  |
| @Range(min=,max=)  | 用来校验数字或字符串的大小必须在指定的min和max内 字符串会转成数字进行比较，如果不是数字校验不通过 允许元素为null |
| @Min()             | 校验数字（包括integer short long int 等）的最小值，不支持小数即double和float 允许元素为null |
| @Max()             | 校验数字（包括integer short long int 等）的最小值，不支持小数即double和float 允许元素为null |
| @Pattern()         | 正则表达式匹配，可用来校验年月日格式，是否包含特殊字符（regexp = "^[a-zA-Z0-9\u4e00-\u9fa5 |

除了@Empty要求字符串不能全是空格，其他的字符串校验都是允许空格的。
message是可以引用常量的，但是如@Size里max不允许引用对象常量，基本类型常量是可以的。
注意大部分规则校验都是允许参数为null，即当不存在这个值时，就不进行校验了



#### 3.2、不常用注解

| 注解 | 说明 |
| ---- | ---- |
| @Null |被注释的元素必须为 null|
|@AssertTrue| 被注释的元素必须为 true|
|@AssertFalse| 被注释的元素必须为 false|
|@DecimalMin(value) |被注释的元素必须是一个数字，其值必须大于等于指定的最小值|
|@DecimalMax(value) |被注释的元素必须是一个数字，其值必须小于等于指定的最大值|
|@Digits (integer, fraction)| 被注释的元素必须是一个数字，其值必须在可接受的范围内|
|@Past| 被注释的元素必须是一个过去的日期|
|@Future |被注释的元素必须是一个将来的日期|
|@Email |被注释的元素必须是电子邮箱地址|



#### 3.3、自定义注解

##### 3.3.1、自定义注解

```
 @Target({ElementType.METHOD, ElementType.FIELD, ElementType.ANNOTATION_TYPE})
    @Retention(RetentionPolicy.RUNTIME)
    @Constraint(validatedBy = EnumCheckValidator.class)
    public @interface EnumCheck {
        /**
         * 是否必填 默认是必填的
         * @return
         */
        boolean required() default true;
        /**
         * 验证失败的消息
         * @return
         */
        String message() default "枚举的验证失败";
        /**
         * 分组的内容
         * @return
         */
        Class<?>[] groups() default {};
    
        /**
         * 错误验证的级别
         * @return
         */
        Class<? extends Payload>[] payload() default {};
    
        /**
         * 枚举的Class
         * @return
         */
        Class<? extends Enum<?>> enumClass();
    
        /**
         * 枚举中的验证方法
         * @return
         */
        String enumMethod() default "validation";
    }
```



##### 3.3.2、实现校验的逻辑

```
public class EnumCheckValidator implements ConstraintValidator<EnumCheck, Object> {

    private static final Logger logger = LoggerFactory.getLogger(EnumCheckValidator.class);

    private EnumCheck enumCheck;

    @Override
    public void initialize(EnumCheck enumCheck) {
        this.enumCheck =enumCheck;
    }

    @Override
    public boolean isValid(Object value, ConstraintValidatorContext constraintValidatorContext) {
        // 注解表明为必选项 则不允许为空，否则可以为空
        if (value == null) {
            return !this.enumCheck.required();
        }

        Boolean result = Boolean.FALSE;
        Class<?> valueClass = value.getClass();
        try {
            //通过反射执行枚举类中validation方法
            Method method = this.enumCheck.enumClass().getMethod(this.enumCheck.enumMethod(), valueClass);
            result = (Boolean)method.invoke(null, value);
            if(result == null){
                return false;
            }
        } catch (Exception e) {
            logger.error("custom EnumCheckValidator error", e);
        }
        return result;
    }
}
```

##### 3.3.3、测试的demo

```
 public enum  Sex{
        MAN("男",1),WOMAN("女",2);
    
        private String label;
        private Integer value;
    
        public String getLabel() {
            return label;
        }
    
        public void setLabel(String label) {
            this.label = label;
        }
    
        public Integer getValue() {
            return value;
        }
    
        public void setValue(Integer value) {
            this.value = value;
        }
    
        Sex(String label, int value) {
            this.label = label;
            this.value = value;
        }
    
        /**
         * 判断值是否满足枚举中的value
         * @param value
         * @return
         */
        public static boolean validation(Integer value){
            for(Sex s:Sex.values()){
                if(Objects.equals(s.getValue(),value)){
                    return true;
                }
            }
            return false;
        }
    }
    
    @EnumCheck(message = "只能选男：1或女:2",enumClass = Sex.class)
    private Integer sex;
```

#### 3.4、分组校验

```
public class User {
    @NotBlank(message = "用户名不能为空",groups = FirstGroup.class)
    private String userName;
    @NotBlank(message = "密码不能为空",groups = SecobdGroup.class)
   }
```

group分别是FirstGroup.class，SecondGroup.class,这两个都是接口，名称随便定义也不需要实现，只是能做区分分组就行；当然不写时它会有个默认的分组Default.class，所有不填写的都会划入这个分组

```
public static void main(String args[]) {
        User user = new User();
        user.setUserName("zhao");
        user.setPassword("123456");
        Set<ConstraintViolation<User>> result = ValidateUtil.validate(user,FirstGroup.class);
        // 抛出检验异常
        if (result.size() > 0) {
            Iterator<ConstraintViolation<User>> it = result.iterator();
            while (it.hasNext()) {
                ConstraintViolation<User> str = it.next();
                System.out.println(str.getMessage() + "\n");
            }
        }
    }
```

代码校验ValidateUtil.validate(user,FirstGroup.class);加了分组FirstGroup()，这样即使分组二不满足条件但也不会校验。如果想2个分组都校验可以把2个分组都带上。例:ValidateUtil.validate(user,FirstGroup.class,SecondGroup.class),这样
添加这两个分组注释的字段会都校验。



### 4、常见问题说明

| 问题               | 解决方案             |
| ------------------ | -------------------- |
| 多级属性校验不生效 | 父级bean上增加@Valid |

