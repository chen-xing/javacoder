### aop联合反射实现mysql与hbase的读写切换

### 1、能够做的事情

+ 开关控制mysql和hbase的读写切换，实时生效
+ 内置监控可以观察hbase的运行性能（刚切过去，性能稳定性有待考究）



### 2、核心的代码

```
package tech.chenxing.aop;

import tech.chenxing.conf.RgSystemConfig;
import tech.chenxing.util.SpringTool;
import lombok.extern.slf4j.Slf4j;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Pointcut;
import org.aspectj.lang.reflect.MethodSignature;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;
import java.time.Duration;
import java.time.Instant;

@Aspect
@Component
@Slf4j
public class MultiDataSourceAspect {

    @Autowired private RgSystemConfig rgSystemConfig;

    /** 定义一个切入点 */
    @Pointcut("@annotation(cn.esign.aop.MultiDataSource)")
    private void multiDataSourceAspect() {}

    @Around("multiDataSourceAspect()")
    public Object methodMonitor(ProceedingJoinPoint pjp) throws Throwable {
        // 获取方法签名
        MethodSignature signature = (MethodSignature) pjp.getSignature();
        // 获取切入方法的对象
        Method method = signature.getMethod();
        // 获取方法上的Aop注解
        MultiDataSource annotation = method.getAnnotation(MultiDataSource.class);
        Class invokeClass = annotation.invokeClass();
        boolean readOperate = annotation.isReadOperate();
        boolean hbaseStable = rgSystemConfig.isHbaseStable();
        boolean hbaseDualWrite = rgSystemConfig.isHbaseDualWrite();
        if (readOperate) {
            if (hbaseStable) {
                return proceed(pjp);
            } else {
                return invokeRdsOperate(pjp, method, invokeClass);
            }
        } else {
            if (hbaseDualWrite) {
                proceed(pjp);
                // 拦截的方法参数
                return invokeRdsOperate(pjp, method, invokeClass);
            } else {
                if (hbaseStable) {
                    return proceed(pjp);
                } else {
                    return invokeRdsOperate(pjp, method, invokeClass);
                }
            }
        }
    }

    /**
     * 执行方法并关注耗时
     *
     * @param pjp
     * @return
     */
    private Object proceed(ProceedingJoinPoint pjp) throws Throwable {
        Object result = null;
        try {
            Long alarmThreshold = rgSystemConfig.getAlarmThreshold();
            Instant start = Instant.now();
            result = pjp.proceed();
            Instant end = Instant.now();
            long cost = Duration.between(start, end).toMillis();
            if (alarmThreshold > 0 && cost > alarmThreshold) {
                log.warn("hbase dal operate cost:{} ms", cost);
            }
        } catch (Throwable ex) {
            boolean hbaseStable = rgSystemConfig.isHbaseStable();
            if (hbaseStable) {
                throw ex;
            } else {
                log.warn("hbase dal error:{}", ex.getMessage());
            }
        }
        return result;
    }

    /**
     * 是否操作老的数据库
     *
     * @param pjp
     * @param method
     * @param invokeClass
     * @return
     * @throws NoSuchMethodException
     * @throws InvocationTargetException
     * @throws IllegalAccessException
     */
    private Object invokeRdsOperate(ProceedingJoinPoint pjp, Method method, Class invokeClass)
            throws NoSuchMethodException, InvocationTargetException, IllegalAccessException {
        Object[] args = pjp.getArgs();
        Class<?>[] methodParamArr = method.getParameterTypes();
        String methodName = method.getName();
        Object clz = SpringTool.getBean(invokeClass);
        Method invokeMethod = clz.getClass().getMethod(methodName, methodParamArr);
        return invokeMethod.invoke(clz, args);
    }
}

```



