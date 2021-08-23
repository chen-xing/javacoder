# 借助jmeter实现自动化执行测试任务

## 1、大致流程

+ 触发测试用例，大约需要执行10分钟
+ 查询执行的commitId的测试用例的执行结果
+ 如果通过了则直接停止测试用例，或beanshell 异常也要停止，否则获取不到结果，无限执行也是没有意义的，否则继续循环执行(设置最大的重试次数)

!["自动化执行流程"](https://trial-cdn.esign.cn/upload/009d16dd-62ce-5cb2-b5af-e7259eff4279!!7-8.png)

## 2、延迟执行request

+ 如果是1和2中间需要停止10分钟，可以在 1和2中间新增一个request-3,并且配置timer的时长
+ 定时器是在每个sampler（采样器）之前执行的，而不是之后。不管这个定时器的位置放在sampler之后，还是之下，它都在sampler之前得到执行。
+ 定时器是有作用域的；当执行一个sampler之前时，所有当前作用域内的定时器都会被执行；
+ 如果希望定时器仅应用于其中一个sampler，则把该定时器作为子节点加入；
+ 如果希望在sampler执行完之后再等待，则可使用取样器里面的测试活动(Test Action)；	
+ 更优雅的实现步骤间的停顿的方案是 **Flow Control Action**			

## 3、提取request的返回值

+ 添加beanshell postprocessor
+ testplan中引入fastjson的jar依赖（或者把jar放置到jmeter的lib的目录下）
+ JMeterContextService.getContext().getThread().stop(); 满足条件，提前中断了测试计划的执行。（try catch中需要捕获异常，异常场景需要提前结束测试计划，以防beanshell 有bug 无限执行）
+ vars.get("test_flag")获取系统变量；
+ vars.put("test_flag","true");设置用户自定义变量；注意的是value需要为字符串

样例：

```
import com.alibaba.fastjson.JSONObject;
import com.alibaba.fastjson.JSONArray;
import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.JSONPath;
import java.util.List;
import org.apache.jmeter.threads.JMeterContextService;
try{
	//获取获取请求的返回值
	String response_data = prev.getResponseDataAsString();
	//日志打印获取请求的返回值
	log.info("---response_data---" + response_data);
	//将返回值转换成JSON对象
	JSONObject data_obj= JSON.parseObject(response_data);  
	log.info("------data_obj--------" + data_obj.toString());
	//获取JSON中data列表
	JSONObject data_object = data_obj.getJSONObject("data");
	log.info("---data_arr---" + data_object);
	//获取Province数组的长度
	boolean flag=data_object.getBoolean("flag");
	if(flag){
		vars.put("test_flag","true");
		Failure=false;//集测通过了，直接终止脚本的执行
		JMeterContextService.getContext().getThread().stop();
	}
	log.info(vars.get("test_flag"));
}catch(Exception e){
	log.info("beanshell failed",e);
	JMeterContextService.getContext().getThread().stop();
}
```



## 4、拓展

测试计划中的元件执行顺序依次为：

配置元件（CSV Data Set Config）-前置处理器-定时器-取样器-后置处理器-断言-监听器