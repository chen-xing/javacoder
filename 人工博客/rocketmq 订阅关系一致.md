### rocketmq 订阅关系一致

### 1、背景

今天上线了一个新的mq的消费逻辑，刚发布上线就收到了mq的告警。消息积压了3000w。瞬间一顿懵逼。但是直觉告诉我肯定和这个新上的业务有关。直接回滚。告警恢复。

### 2、订阅关系一致

订阅关系一致指的是同一个消费者 Group ID 下所有 Consumer 实例的处理逻辑必须完全一致。一旦订阅关系不一致，消息消费的逻辑就会混乱，甚至导致消息丢失。本文提供订阅关系不一致的示例代码，帮助您顺畅地订阅消息。

#### 2.1、背景信息
消息队列 RocketMQ 版里的一个消费者 Group ID 代表一个 Consumer 实例群组。对于大多数分布式应用来说，一个消费者 Group ID 下通常会挂载多个 Consumer 实例。

由于消息队列 RocketMQ 版的订阅关系主要由 Topic + Tag 共同组成，因此，保持订阅关系一致意味着同一个消费者 Group ID 下所有的实例需在以下两方面均保持一致：

+ 订阅的 Topic 必须一致
+ 订阅的 Topic 中的 Tag 必须一致

#### 2.2、正确订阅关系图片示例
多个 Group ID 订阅了多个 Topic，并且每个 Group ID 里的多个消费者实例的订阅关系保持了一致。

![正确订阅关系图片示例](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/0739885751/p69113.png)

#### 2,3、错误订阅关系图片示例
单个 Group ID 订阅了多个 Topic，但是该 Group ID 里的多个消费者实例的订阅关系并没有保持一致。

![错误订阅关系图片示例](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/1739885751/p69117.png)



### 3、问题思考

这个问题的根因是2个不同的服务，因为 Group ID可以自由指定，正好撞车了而引发的。既然 Group ID这个概念这么重要，那么最好这个id有一个内部的机制来保证唯一性，而非随意指定。springclound可以取服务的serviceId来充当。