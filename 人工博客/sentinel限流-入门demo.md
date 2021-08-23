### sentinel限流-入门demo

### 摘要

熔断限流和最终一致性是保护系统平稳运行的两把神器。

Hystrix和Sentinel这两个开源组件都是不错的选择，但我更看好Sentinel。



### 关键字

sentine，熔断限流



### 1、pom引入

```
<dependency>
	<groupId>com.alibaba.csp</groupId>
	<artifactId>sentinel-core</artifactId>
	<version>1.8.0</version>
</dependency>
```



### 2、定义资源并初始化规则

```
List<FlowRule> rules = new ArrayList<>();
FlowRule rule = new FlowRule();
rule.setResource("94rg");
rule.setGrade(RuleConstant.DEGRADE_GRADE_EXCEPTION_COUNT);
// Set limit QPS to 20.
rule.setCount(1);
rules.add(rule);
FlowRuleManager.loadRules(rules);
```

+ 资源一般是一个接口、一个方法，可以类比一个分布式锁的key
+ 规则目前支持的有RT、QPS、异常比（选择模式以及对应的阀值）
+ 添加到FlowRuleManager进行管理

### 3、开启限流

+ 关键的限流逻辑代码就以下几句

  ```
  Entry entry = null;
  try {
  	entry = SphU.entry("94rg");
  	/*您的业务逻辑 - 开始*/
  	System.out.println("hello world");
  	/*您的业务逻辑 - 结束*/
  } catch (BlockException e1) {
  	/*流控逻辑处理 - 开始*/
  	System.out.println("block!");
  	/*流控逻辑处理 - 结束*/
  } catch (Exception e) {
  	e.printStackTrace();
  } finally {
  	if (entry != null) {
  		entry.exit();
  	}
  }
  ```

  

### 4、完整demo

```
package tech.chenxing;

import com.alibaba.csp.sentinel.Entry;
import com.alibaba.csp.sentinel.SphU;
import com.alibaba.csp.sentinel.slots.block.BlockException;
import com.alibaba.csp.sentinel.slots.block.RuleConstant;
import com.alibaba.csp.sentinel.slots.block.flow.FlowRule;
import com.alibaba.csp.sentinel.slots.block.flow.FlowRuleManager;
import lombok.extern.slf4j.Slf4j;
import org.junit.jupiter.api.BeforeAll;
import org.junit.jupiter.api.Test;
import org.springframework.test.context.event.annotation.BeforeTestClass;

import java.util.ArrayList;
import java.util.List;

@Slf4j
public class SentinelTest {
    @BeforeAll
    public static void initRules(){
        List<FlowRule> rules = new ArrayList<>();
        FlowRule rule = new FlowRule();
        rule.setResource("94rg");
        rule.setGrade(RuleConstant.DEGRADE_GRADE_EXCEPTION_COUNT);
        // Set limit QPS to 20.
        rule.setCount(1);
        rules.add(rule);
        FlowRuleManager.loadRules(rules);
    }

    @Test
    public void limitRequestTest(){
        int i=0;
        while (true) {
            i++;

            Entry entry = null;
            try {
                if(i%7==0){
                    throw new Exception("异常数过多");
                }
                entry = SphU.entry("94rg");
                /*您的业务逻辑 - 开始*/
                System.out.println("hello world");
                /*您的业务逻辑 - 结束*/
            } catch (BlockException e1) {
                /*流控逻辑处理 - 开始*/
                System.out.println("block!");
                try {
                    Thread.sleep(200);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                /*流控逻辑处理 - 结束*/
            } catch (Exception e) {
                e.printStackTrace();
            } finally {
                if (entry != null) {
                    entry.exit();
                }
            }
        }
    }
}
```

