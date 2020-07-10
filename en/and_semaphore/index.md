# and 信号量



<!--more-->

## 问题引出

信号量可以很好的解决单个资源的互斥访问。

在有些应用中，进程需要先获得两个或更多共享资源后才能执行其任务，由此产生进程对多个资源的互斥访问。

比如p1、p2共享两个全局变量，r1\r2，p1先P(mutex1),p2再P(mutex2),然后两个分别再申请另一个变量，这时候，两个进程都会阻塞，无法再继续执行。

## And信号量

and信号量同步机制就是要解决上述问题，其基本思想是将进程在整个运行期间所需要的所有临界资源一次性全部分配给进程，待进程使用完成后再一起释放。

只要有一个资源不能满足进程的要求，其他资源也都不会分配，因此再P操作上增加了and条件，所以称作and信号量。

## and信号量 VS 普通信号量

代码上：
普通信号量的P操作中只需要判断单个资源的数量是否满足进程要求。
and信号量的P操作中使用&&操作符判断多个资源的数量是否满足进程要求。

使用上：
普通信号量适用于申请单个资源。
and信号量适用于一次性申请多个资源，防止死锁发生。

## 应用

* 用and信号量解决哲学家进餐问题
* 用and信号量解决生产者-消费者问题