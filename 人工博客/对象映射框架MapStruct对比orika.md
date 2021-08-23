### 对象映射框架MapStruct对比orika

### 1、各大对象映射框架性能对比

| 工具             | 实现方式          | 缺点                                                         | 说明                                                         |
| ---------------- | ----------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| mapstruct        | getter/setter方法 | 需要了解注解和配置项语法                                     | JSR269注解处理器在编译期自动生成Java<br/>Bean转换代码，支持可配置化，扩展性强 |
| orika            | 动态生成字节码    | 首次调用耗时较久,性能适中                                    | 采用javassist类库生成Bean映射的字节码，之后直接加载执行生成的字节码文件 |
| Spring BeanUtils | 反射机制          | 不支持名称相同但类型不同的属性转换                           |                                                              |
| Apache BeanUtils | 反射机制          | 需要处理编译期异常，性能最差                                 |                                                              |
| dozer            | 反射机制          | 性能差                                                       | 使用reflect包下Field类的set(Object<br/>obj, Object value)方法进行属性赋值 |
| BeanCopier       | 反射机制          | \1. BeanCopier只拷贝名称和类型都相同的属性。即便基本类型与其对应的包装类型也不能相互转换; | 使用ASM的MethodVisitor直接编写各属性的get/set方法            |

> 就性能而言：**mapstruct**性能无疑是是最高的，接下来依次是Spring BeanUtils>orika>BeanCopier>dozer>apache BeanUtils



### 2、MapStruct的使用

#### 2.1、引入pom

```
<dependency>
	<groupId>org.mapstruct</groupId>
	<artifactId>mapstruct</artifactId>
	<version>1.3.0.Final</version>
	<scope>provided</scope>
</dependency>
```



#### 2.2 、单个bean映射

```
import org.mapstruct.Mapper;
import org.mapstruct.Mapping;

@Mapper(componentModel = "spring")
public interface OrderConvert {

    @Mapping(source = "id", target = "orderId")
    @Mapping(source = "createTime", target = "orderTime", dateFormat = "yyyy-MM-dd HH:mm:ss")
    OrderDTO from(Order order);

}
```

测试代码

```
@Test
public void test() {
    Order order = Order.builder()
        .id(123L)
        .buyerPhone("13707318123")
        .buyerAddress("中电软件园")
        .amount(10000L)
        .payStatus(1)
        .createTime(LocalDateTime.now())
        .build();

    OrderConvert orderConvert = Mappers.getMapper(OrderConvert.class);
    OrderDTO orderDTO = orderConvert.from(order);

    System.out.println("order:    " + order);
    System.out.println("orderDTO: " + orderDTO);
}
```

#### 2.3、多个bean的映射

```
@Mapper(componentModel = "spring")
public interface GoodInfoConvert {

    /** Long => String 隐式类型转换 */
    @Mapping(source = "good.id", target = "goodId")
    /** 属性名不同， */
    @Mapping(source = "type.name", target = "typeName")
    /** 属性名不同 */
    @Mapping(source = "good.title", target = "goodName")
    /** 属性名不同 */
    @Mapping(source = "good.price", target = "goodPrice")
    GoodInfoDTO from(GoodInfo good, GoodType type);

}
```



#### 2.4、参数的含义映射

```
import org.mapstruct.Mapper;
import org.mapstruct.Mapping;

import java.time.LocalDate;
import java.time.LocalDateTime;

@Mapper(imports = {CustomMapping.class})
public interface StudentConvert {

    @Mapping(source = "id", target = "studentId")
    @Mapping(source = "name", target = "studentName")
    @Mapping(source = "age", target = "age")
    @Mapping(target = "ageLevel", expression = "java(CustomMapping.ageLevel(student.getAge()))")
    @Mapping(target = "sexName", expression = "java(CustomMapping.sexName(student.getSex()))")
    @Mapping(source = "admissionTime", target = "admissionDate", dateFormat = "yyyy-MM-dd")
    StudentDTO from(Student student);

    default LocalDate map(LocalDateTime time) {
        return time.toLocalDate();
    }

}
```

自定义的映射类

```
public class CustomMapping {

    static final String[] SEX = {"女", "男", "未知"};

    public static String sexName(Integer sex) {

        if (sex < 0 && sex > 2){
            throw new IllegalArgumentException("invalid sex: " + sex);
        }
        return SEX[sex];
    }

    public static String ageLevel(Integer age) {
        if (age < 18) {
            return "少年";
        } else if (age >= 18 && age < 30) {
            return "青年";
        } else if (age >= 30 && age < 60) {
            return "中年";
        } else {
            return "老年";
        }
    }

}
```

### 3、orika的使用

### 3.1 引入pom

```
<dependency>
	<groupId>ma.glasnost.orika</groupId>
	<artifactId>orika-core</artifactId>
	<version>1.5.2</version>
</dependency>
```



#### 3.2、初始化实例

```
package tech.chenxing.deploy.configuration;

import ma.glasnost.orika.MapperFactory;
import ma.glasnost.orika.impl.DefaultMapperFactory;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class MapperConfig {
    @Bean
    public MapperFactory mapperFactory() {
        return new DefaultMapperFactory.Builder().build();
    }
}

```



#### 3.3、配置映射对象

定义转换器

```
public class FlowDocDOConverter extends CustomMapper<FlowDoc, FlowDocBean> {
    @Autowired private DocInfoMapperExt docInfoMapperExt;

    @Override
    public void mapAtoB(FlowDoc flowDocDO, FlowDocBean flowDocBean, MappingContext context) {
        if (StringUtils.isNotBlank(flowDocDO.getDocUuid())) {

            DocInfo docInfoDO = docInfoMapperExt.getByUUID(flowDocDO.getDocUuid());
            ValidationUtil.assertNull(
                    docInfoDO,
                    new BizFlowManagerException(
                            BizFlowManagerBaseResultCodeEnum.FLOW_NOT_EXISTS,
                            flowDocDO.getDocUuid()));
            flowDocBean.setId(docInfoDO.getId());
            flowDocBean.setFileId(docInfoDO.getFileId());
            flowDocBean.setName(docInfoDO.getName());
            flowDocBean.setDocPassword(docInfoDO.getDocPassword());
            flowDocBean.setDocSource(docInfoDO.getDocSource());
            flowDocBean.setEncryption(docInfoDO.getEncryption());
            flowDocBean.setGmtCreate(flowDocDO.getGmtCreate());
            flowDocBean.setGmtModified(flowDocDO.getGmtModified());
        }
    }
}
```



注入转换器

```
package tech.chenxing.deploy.configuration;

import ma.glasnost.orika.MapperFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

import javax.annotation.PostConstruct;


@Component
public class MapperInit {
    @Autowired private MapperFactory mapperFactory;
    @Autowired private FlowDocDOConverter flowDocDOConverter;


    @PostConstruct
    public void init() {
        // 注册文档映射方式
        mapperFactory
                .classMap(User.class, UserDTO.class)
                .field("docUuid", "docId")
                .byDefault() // 剩余的字段映射
                .customize(flowDocDOConverter)
                .register();
    }
}

```

代码使用

```
public xxx queryuser() {
        List<User> userList =
                userMapperExt.queryUser();
        List<UserDTO> userDTOList =
                mapperFactory.getMapperFacade().mapAsList(userList, UserDTO.class);
        QueryFlowDocResult queryFlowDocResult = new QueryFlowDocResult();
    }
```