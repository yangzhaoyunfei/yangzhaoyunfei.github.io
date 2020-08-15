# 如何保证消息的顺序性


<!--more-->
## Case

一个 mysql binlog 同步的系统，日同步数据要达到上亿. 用来将数据从一个 mysql 库原封不动地同步到另一个 mysql 库里面去（mysql -> mysql）。

你在 mysql 里增删改一条数据，对应出来了增删改 3 条 binlog 日志，接着这三条 binlog 发送到 MQ 里面，再消费出来依次执行，必须保证他们的顺序. 原来是：增加、修改、删除；顺序错了给执行成删除、修改、增加, mysql执行得到的数据就是错的。

## Scenarios

### RabbitMQ

一个 queue，多个 consumer.

比如，生产者向 RabbitMQ 里发送了三条数据，顺序依次是 data1/data2/data3，压入的是 RabbitMQ 的一个内存队列。有三个消费者分别从 MQ 中消费这三条数据中的一条，结果消费者2先执行完操作，把 data2 存入数据库，然后是 data1/data3。这不明显乱了。

![pop](/images/working/mq21.jpg)

### Kafka

一个 topic，一个 partition，一个 consumer(内部多线程).

生产者在写的时候，其实可以指定一个 key，比如说我们指定了某个订单 id 作为 key，那么这个订单相关的数据，一定会被分发到同一个 partition 中去，而且这个 partition 中的数据一定是有顺序的。消费者从 partition 中取出来数据的时候，也一定是有顺序的。到这里，顺序还是 ok 的，没有错乱。接着，我们在消费者里可能会搞**多个线程来并发处理消息**。因为如果消费者是单线程消费处理，而处理比较耗时的话，比如处理一条消息耗时几十 ms，那么 1 秒钟只能处理几十条消息，这吞吐量太低了。而多个线程并发跑的话，顺序可能就乱掉了。

![pop](/images/working/mq22.jpg)

## Solution

### RabbitMQ

二选一:

* 拆分成多个 queue, 每个 queue 一个 consumer;
* 一个 queue, 一个 consumer, 该 consumer 内部用内存队列做排队, 然后分发给底层不同的 worker 来处理。

![pop](/images/working/mq23.jpg)

### Kafka

* 一个 topic，一个 partition，一个 consumer，内部单线程消费，单线程吞吐量太低，一般不会用这个。
* 写 N 个内存 queue，具有相同 key 的数据都到同一个内存 queue；然后对于 N 个线程，每个线程分别消费一个内存 queue 即可，这样就能保证顺序性。

![pop](/images/working/mq24.jpg)

## 总结

分析:其实并非所有的公司都有这种业务需求，但是还是对这个问题要有所复习。

回答:针对这个问题，通过某种算法，将需要保持先后顺序的消息放到同一个消息队列中(kafka中就是partition,rabbitMq中就是queue)。然后只用一个消费者去消费该队列。

有的人会问:那如果为了吞吐量，有多个消费者去消费怎么办？

这个问题，没有固定回答的套路。比如我们有一个微博的操作，发微博、写评论、删除微博，这三个异步操作。如果是这样一个业务场景，那只要重试就行。

比如你一个消费者先执行了写评论的操作，但是这时候，微博都还没发，写评论一定是失败的，等一段时间。等另一个消费者，先执行写评论的操作后，再执行，就可以成功。

总之，针对这个问题，我的观点是保证入队有序就行，出队以后的顺序交给消费者自己去保证，没有固定套路。

## Reference

* [https://zhuanlan.zhihu.com/p/60166828](https://zhuanlan.zhihu.com/p/60166828)
* [https://zhuanlan.zhihu.com/p/92622338](https://zhuanlan.zhihu.com/p/92622338)
