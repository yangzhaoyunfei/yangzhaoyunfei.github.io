# message queue


<!--more-->
## 什么是消息队列

我们可以把消息队列比作是一个存放消息的容器, 当我们需要使用消息的时候可以取出消息供自己使用。消息队列是分布式系统中的重要组件, 使用消息队列主要是为了**通过异步处理提高系统性能和削峰、降低系统耦合性**。

## 为什么要用消息队列

使用消息队列主要有以下好处：
1. 解耦
1. 异步
1. 削峰

### 解耦

![pop](/images/working/mq3.jpg "利用消息队列实现事件驱动结构")

* 传统模式

    系统间耦合性太强，如果系统A在代码中直接调用系统B和系统C的代码，将来D系统接入，系统A还需要修改代码，过于麻烦！

* 中间件模式

    将消息写入消息队列，需要消息的系统自己从消息队列中订阅，从而系统A不需要做任何修改。如果模块之间不存在直接调用，那么新增模块或者修改模块就对其他模块影响较小.

### 异步

![pop](/images/working/mq13.jpg)

* 传统模式

    一些非必要的业务逻辑以同步的方式运行，太耗费时间。
    
![pop](/images/working/mq14.jpg)

* 中间件模式

    将消息写入消息队列，非必要的业务逻辑以异步的方式运行，加快响应速度

### 削峰

![pop](/images/working/mq15.jpg)

* 传统模式

    并发量大的时候，所有的请求直接怼到数据库，造成数据库连接异常
    
![pop](/images/working/mq16.jpg)

* 中间件模式

    系统A慢慢的按照数据库能处理的并发量，从消息队列中慢慢拉取消息。在生产中，这个短暂的高峰期积压是允许的。

## 使用消息队列带来的问题

* 系统可用性降低
    
    系统可用性在某种程度上降低，在加入MQ之前，你不用考虑消息丢失或者说MQ挂掉等等的情况，但是，引入MQ之后你就需要去考虑了！
    
* 系统复杂性提高
    
    加入MQ之后，你需要保证消息没有被重复消费、处理消息丢失的情况、保证消息传递的顺序性等等问题！
    
* 一致性问题
    
    消息队列可以实现异步，消息队列带来的异步确实可以提高系统响应速度。但是，万一消息的真正消费者并没有正确消费消息怎么办？这样就会导致数据不一致的情况了!

##  JMS VS AMQP

| item | JMS |AMQP |
| --- | ----------- |----------- |
| 定义 | Java API规范 | 协议 |
| 跨语言 | 否 | 是 |
| 跨平台 | 否 | 是 |
| 支持模式 | 1. p2p; <br> 2. pub/sub;| 1. direct exchange;<br>2. fanout exchange;<br>3. topic exchange;<br>4. headers exchange;<br>5. system exchange; <br>```本质上讲,后4种与jms的pub/sub模式没有太大差别``` |
| 支持消息类型 | 支持多种类型 | byte[](二进制) |

总结:

* AMQP 为消息定义了线路层（wire-level protocol）的**协议**，而**JMS所定义的是API规范**。在 Java 体系中，多个client均可以通过JMS进行交互，不需要应用修改代码，但是其对跨平台的支持较差。而AMQP天然具有跨平台、跨语言特性。
* JMS 支持TextMessage、MapMessage 等复杂的消息类型；而 AMQP 仅支持 byte[] 消息类型（复杂的类型可序列化后发送）。
由于Exchange 提供的路由算法，AMQP可以提供多样化的路由方式来传递消息到消息队列，而 JMS 仅支持 队列 和 主题/订阅 方式两种。
* 使用JMS需要相应的API实现方, 如 ActiveMQ; 使用AMQP需要选择基于AMQP协议实现的产品, 如 RabbitMQ;

## 常见消息队列对比

| 特性 | ActiveMQ | RabbitMQ | Kafka | RocketMQ |
| --- | --- | --- | --- | --- |
| producer-comsumer | 支持 | 支持 | 支持 | 支持 |
| publish-subscribe | 支持 | 支持 | 支持 | 支持 |
| request-reply | 支持 | 支持 | - | 支持 |
| API完备性 | 高 | 高 | 高 | 低（静态配置） |
| 多语言支持 | 支持，JAVA优先 | 语言无关 | 支持，JAVA优先 | 支持 |
| 单机呑吐量 | 万级 | 万级 | 十万级 | 十万级 |
| 消息延迟 | - | 微秒级 | 毫秒级 | - |
| 可用性 | 高（主从） | 高（主从） | 非常高（分布式） | 高 |
| 消息丢失 | - | 低 | 理论上不会丢失 | - |
| 消息重复 | - | 可控制 | 理论上会有重复 | - |
| 文档的完备性 | 高 | 高 | 高 | 中 |
| 提供快速入门 | 有 | 有 | 有 | 无 |
| 首次部署难度 | - | 低 | 中 | 高 |

注: - 表示尚未查找到准确数据

## 消息队列选型

从上面对比可知:
1. 中小型软件公司，建议选RabbitMQ
    
    一方面，erlang语言天生具备高并发的特性，而且他的管理界面用起来十分方便。  
    正所谓，成也萧何，败也萧何！他的弊端也在这里，虽然RabbitMQ是开源的，然而国内有几个能定制化开发erlang的程序员呢？  
    所幸，RabbitMQ的社区十分活跃，可以解决开发过程中遇到的bug，这点对于中小型公司来说十分重要。  
    不考虑rocketmq和kafka的原因是，一方面中小型软件公司不如互联网公司，数据量没那么大，选消息中间件，应首选功能比较完备的，所以kafka排除。  
    不考虑rocketmq的原因是，rocketmq是阿里出品，如果阿里放弃维护rocketmq，中小型公司一般抽不出人来进行rocketmq的定制化开发，因此不推荐。
    
1. 大型软件公司，根据具体使用在rocketMq和kafka之间二选一
    
    一方面，大型软件公司，具备足够的资金搭建分布式环境，也具备足够大的数据量。  
    针对rocketMQ,大型软件公司也可以抽出人手对rocketMQ进行定制化开发，毕竟国内有能力改JAVA源码的人，还是相当多的。  
    至于kafka，根据业务场景选择，如果有日志采集功能，肯定是首选kafka了。具体该选哪个，看使用场景。
    
## 如何保证消息队列服务的高可用性

高可用的根本解决方案: 冗余;
以rcoketMQ为例，他的集群就有多master 模式、多master多slave异步复制模式、多 master多slave同步双写模式。

![pop](/images/working/mq17.jpg "多master多slave模式部署架构图")

## 如何防止消息重复消费

即保证消息队列的幂等性, 这个需要根据具体的业务场景来分析并给出解决方案.

### 重复消费产生的原因

正常情况下，消费者在消费完毕一个消息后，会发送一个确认信息给消息队列，消息队列就知道该消息被消费了，就会将该消息从消息队列中删除。只是不同的消息队列发送的确认信息形式不同.例如RabbitMQ是发送一个ACK确认消息，RocketMQ是返回一个CONSUME_SUCCESS成功标志，kafka是提交消息的offset.

无论哪种消息队列，造成重复消费的原因都是类似的: 

**因为网络传输等等故障，确认信息没有传送到消息队列，导致消息队列不知道已经消费过该消息了，再次将该消息分发给其他的消费者**。

### 解决方案

需要结合具体的业务场景.
1. 拿到消息做数据库的insert操作
    
    给每个消息做一个唯一主键，那么就算出现重复消费的情况，就会导致主键冲突，避免数据库出现脏数据。
    
1. 拿到消息做redis的set的操作

    不用解决。因为你无论set几次结果都是一样的，set操作本来就算幂等操作。
    
1. 其他场景

    使用第三方来做消费记录。以redis为例，给消息分配一个全局id，只要消费过该消息，将以K-V形式写入redis。那消费者开始消费前，先去redis中查询有没消费记录即可。
    
## 如何保证消息传输的可靠性

从三个角度分析:
1. 生产者丢数据;
1. 消息队列丢数据;
1. 消费者丢数据;

### RabbitMQ

#### 生产者丢数据

RabbitMQ 提供 transaction和confirm模式来确保生产者不丢消息。

transaction机制就是说，发送消息前，开启事务(channel.txSelect())，然后发送消息，如果发送过程中出现什么异常，事务就会回滚(channel.txRollback())，如果发送成功则提交事务(channel.txCommit())。
缺点就是吞吐量下降了。因此，按照博主的经验，生产上用confirm模式的居多。
一旦channel进入confirm模式，所有在该信道上面发布的消息都将会被指派一个唯一的ID(从1开始)
一旦消息被投递到所有匹配的队列之后，rabbitMQ就会发送一个Ack给生产者(包含消息的唯一ID)
这就使得生产者知道消息已经正确到达目的队列了.如果rabiitMQ没能处理该消息，则会发送一个Nack消息给你，你可以进行重试操作。

处理Ack和Nack的代码如下所示:
```java
channel.addConfirmListener(new ConfirmListener()){
    @Override
    public void handleNack(long deliveryTag, boolean multiple) throws IOException {
        System.out.println("nack: deliveryTag = " + deliveryTag + " multiple: " + multiple);    
    }
    
    @Override
    public void handleAck(long deliveryTag, boolean multiple) throws IOException {
        System.out.println("ack: deliveryTag = " + deliveryTag + " multiple: " + multiple);    
    }

});
```

#### 消息队列丢数据

开启持久化磁盘的配置。

这个持久化配置可以和防止生产者丢数据的confirm机制配合使用，你可以在消息持久化磁盘后，再给生产者发送一个Ack信号。
这样，如果消息持久化磁盘之前，rabbitMQ挂了，那么生产者收不到Ack信号，生产者会自动重发。

持久化只需:

* 将queue的持久化标识durable设置为true, 则代表是一个持久的队列
* 发送消息的时候将deliveryMode=2

这样设置以后，rabbitMQ就算挂了，重启后也能恢复数据.

#### 消费者丢数据

手动确认消息.

消费者丢数据一般是因为采用了自动确认消息模式。
这种模式下，消费者会自动确认收到信息。这时rahbitMQ会立即将消息删除，如果后续消费者出现异常而没能处理该消息，就会丢失该消息。

### Kafka

![pop](/images/working/mq18.jpg)

Producer在发布消息到某个Partition时，先通过ZooKeeper找到该Partition的Leader, 然后无论该Topic的Replication Factor为多少（也即该Partition有多少个Replica），Producer只将该消息发送到该Partition的Leader。Leader会将该消息写入其本地Log。每个Follower都从Leader中pull数据。

#### 生产者丢数据

在kafka生产中，基本都有一个leader和多个follwer。follwer会去同步leader的信息。针对这种情况，做如下配置:

* 在producer端设置acks=all。这个配置保证了，follwer同步完成后，才认为消息发送成功。
* 在producer端设置retries=MAX，一旦写入失败，这无限重试

#### 消息队列丢数据

消息队列丢数据的情况，无外乎就是，数据还没同步，leader就挂了，这时zookpeer会将其他的follwer切换为leader,那数据就丢失了。针对这种情况，做如下配置:

* replication.factor 参数大于1，即要求每个partition必须有至少2个副本
* min.insync.replicas 参数大于1，即要求一个leader至少感知到有至少一个follower还跟自己保持联系

这两个配置加上上面生产者的配置联合起来用，基本可确保kafka不丢数据

#### 消费者丢数据

这种情况一般是自动提交了offset(```offset：指的是kafka的topic中的每个消费组消费的下标。```)，kafka以为你处理好了, 然后你处理过程中程序挂了。

简单的来说就是一条消息对应一个offset下标，每次消费数据的时候如果提交offset，那么下次消费就会从提交的offset加一那里开始消费。比如一个topic中有100条数据，我消费了50条并且提交了，那么此时的kafka服务端记录提交的offset就是49(offset从0开始)，那么下次消费的时候offset就从50开始消费。

改为手动提交即可.

## 如何保证消息的顺序性

通过某种算法，将需要保持先后顺序的消息放到同一个消息队列中(kafka中就是partition,rabbitMq中就是queue)。然后只用一个消费者去消费该队列。

如果为了吞吐量，有多个消费者去消费怎么办？这种场景需要具体业务具体分析. 针对这个问题，我的观点是保证入队有序就行，出队以后的顺序交给消费者自己去保证，没有固定套路。

后续文章会介绍两个特定场景下的保证消息顺序性的解决方案, 供读者参考.

## Reference
* [https://zhuanlan.zhihu.com/p/52773169](https://zhuanlan.zhihu.com/p/52773169)
* [https://zhuanlan.zhihu.com/p/84007327](https://zhuanlan.zhihu.com/p/84007327)
