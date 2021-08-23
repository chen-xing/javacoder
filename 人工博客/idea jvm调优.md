# idea jvm调优

## 1、背景

idea作为一个高频使用的java IDE。性能的好坏,影响着开发的心情。工欲善其事必先利其器。

## 2、优化后的参数
优化后的感觉是**拙匠常怪工具差**。明明可以流畅的编码，为啥之前选择的是默默忍受，不去改变。真的是流畅了很多。

当前电脑的配置是 windows i5 8核16G

```
# custom IntelliJ IDEA VM options

# 堆大小，按常规操作，设成相同的，避免自动扩容
-Xms1536m
-Xmx1536m
# 年轻代大小，Sun推荐设置为堆大小的3/8
-Xmn576m
# 在JVM启动时即预初始化堆中的所有页，能够快速利用
-XX:+AlwaysPreTouch

# 设置一个较大的元空间初始值，避免频繁GC扩容
-XX:MetaspaceSize=256m
# 元空间最大默认不限制，设一个值保护一下
-XX:MaxMetaspaceSize=768m

# 启用G1 GC
# -XX:+UseG1GC

# 启用CMS GC
-XX:+UseConcMarkSweepGC
# CMS并行标记，降低标记阶段停顿时间
-XX:+CMSParallelRemarkEnabled
# 重新标记前先执行一次新生代GC
-XX:+CMSScavengeBeforeRemark
# 触发CMS GC的堆内存占用比例，调大点以降低GC频率
-XX:CMSInitiatingOccupancyFraction=75
-XX:+UseCMSInitiatingOccupancyOnly

# 对象晋升到老年代的年龄，默认15。根据观察，对IDEA来说设成10就足够了
-XX:MaxTenuringThreshold=10

# 压缩普通对象指针
-XX:+UseCompressedOops

# 指定服务器版JIT编译器，其实不用写，默认已经是了
-server
# JIT代码缓存的大小，默认是240M
-XX:ReservedCodeCacheSize=360M
# 打开JIT分层编译，默认是开启的了
-XX:+TieredCompilation
# 每MB堆空间中的软引用能够存活的近似毫秒数
-XX:SoftRefLRUPolicyMSPerMB=50

# OOM时输出堆dump转储文件
-XX:+HeapDumpOnOutOfMemoryError
# 禁止把某些异常的stack trace优化掉，防止信息被吃了找不到问题
-XX:-OmitStackTraceInFastThrow
# 禁用字节码验证。IDEA的代码足够可靠，不用验证
-Xverify:none
# 启用断言机制（enable assertion）
-ea

-Dfile.encoding=UTF-8
-Dsun.io.useCanonCaches=false
-Djava.net.preferIPv4Stack=true
-Djdk.http.auth.tunneling.disabledSchemes=""

-XX:ErrorFile=$USER_HOME/java_error_in_idea_%p.log
-XX:HeapDumpPath=$USER_HOME/java_error_in_idea.hprof
-javaagent:D:\software\JetBrains\IntelliJ IDEA 2019.2\bin\jetbrains-agent.jar
```

## 3、cutom vm options和idea.exe.vmoptions的区别

+ idea.exe.vmoptions是开发工具自带的，不建议修改，因为升级的时候会进行覆盖
+ cutom vm options是用户自定义的，是留给使用者个性化配置的。默认保存在用户目录下的 .IntelliJIdea2019.2/config
+ 实际操作是idea.exe.vmoptions是全局的配置，cutom vm options会对定义的配置进行覆盖



## 4、修改配置后无法启动

报错的信息如下：

**MaxJavaStackTraceDepth=-1 is outside the allowed range**,本质是配置文件的格式不正确或包含了不能被识别的属性。

实际上的 -XX:+UseParNewGC:设置年轻代为多线程收集 这个属性被废弃了。

![jdk10以上UseParNewGC被废弃了](https://oss.94rg.com/figure_bed/20210802181021.png-94rg002)

idea自带的jdk是jdk11,所以是不包含这个属性的。与表象是一致的。
![idea自带的jdk是jdk11](https://oss.94rg.com/figure_bed/20210802181252.png-94rg002)


idea jvm调优，MaxJavaStackTraceDepth=-1 is outside the allowed range

还在使用idea的默认jvm参数吗？那么是时候动手优化一下你的idea了，体验一下飞一般的感觉。