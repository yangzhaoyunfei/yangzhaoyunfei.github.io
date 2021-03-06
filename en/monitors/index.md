# 管程(Monitors)

<!--more-->


## 引入原因

信号量机制的缺点：进程自备同步操作，P(S)、V(S)操作大量分散在各个进程中，还要仔细安排P(S)的顺序，不易管理，易发生死锁。1974年和1977年，Hore和Hansen提出了管程。

引入管程的目的：
1. **把分散在各进程中的临界区集中起来进行管理**；
1. 防止进程的非法同步操作；
1. 便于利用高级语言来编程，便于验证程序正确性；

## 概念

管程是由局限于自己的若干共享变量(数据结构)及其说明，和所有访问这些共享变量的过程所组成的**软件模块**。

基本思想：把信号量及其操作原语封装在一个对象中，即将共享资源及其操作集中在一个模块中。管程可以使用函数库的形式实现，众多语言中都已经实现了管程。

简单地说管程就是一个概念模型，任何语言都可以实现。

## 特点

1. 局限于管程的共享变量(数据结构)只能被管程中的过程访问；
1. 一个进程通过调用管程中的任一过程进入管程；
1. 任何时候只能有一个进程在管程中执行，调用管程的其他进程将被挂起，以等待管程变为可用，即管程有效地实现互斥。**进入管程的互斥机制由编译器负责，通常使用信号量**。
1. 管程可被系统范围内的进程互斥访问，属于共享资源。

## 组成结构

1. **局部共享变量和条件变量**组成管程内的数据结构；
1. 对数据结构进行操作的一组过程
1. 对数据结构进行初始化的语句
1. 一个互斥锁(由编译器添加)

在管程的简单实现中，编译器为每个管程对象自动加入一把私有的互斥锁。该互斥锁初始状态为解锁，在管程的每个公共过程的入口给该互斥锁加锁，在管程的每个公共过程的出口给该互斥锁解锁。

### 条件变量

条件变量(以c为例)是管程内的一种数据结构，只能通过两个原语来操作它：

* wait(c)：调用进程被阻塞并移入与条件变量c相关的队列中，并释放管程，直到另一个进程在该条件变量c上执行signal()唤醒等待进程并将其移出c的队列。
* signal(c)：如果存在其他进程由于执行wait(c)而被阻塞，便释放一个；如果没有进程在等待，那么信号被丢弃而不会产生任何效果。

条件变量与P、V操作中的信号量的区别：

条件变量是一种信号量，但不是P、V操作中纯粹的计数信号量，不能累计数目，仅仅起到维护等待队列的作用。因此在使用条件变量x时，通常需要定义一个与之配套使用的整型变量x-count用于记录x所维护的等待队列中的进程数。

### 为什么要有条件变量

首先，入口等待队列及互斥锁只能保证进程对临界资源的互斥访问，无法解决进程同步(时序)的问题。于是引入了条件变量及其对应的队列。并为条件变量提供了wait()、signal()原语，使得可以实现进程的阻塞和唤醒，从而提供进程同步的机制。

### 管程入口处的等待队列

管程是互斥进入的，所以当一个进程试图进入一个已被占有的管程时它应该在管程的入口处等待，因而，管程的入口处应当有一个进程等待队列，称为入口等待队列。

### 管程内的资源等待队列

管程是用于管理资源的，当**已进入管程的进程**因资源被占用等原因不能继续执行时应使其等待，即将等待资源的进程加入资源等待队列，该队列由条件变量维护。资源等待队列可以有多个，每种资源一个队列。

### 管程内的紧急等待队列

进入管程中的进程，执行唤醒signal操作后(如P唤醒Q，注意：signal操作可能不在管程过程的最后，看实现管程者是如何编写的)，就有两个进程活跃在管程中，由于管程是互斥使用的，因此必须将其中一个进程挂起：

1. P等待Q继续，直到Q退出或等待；
1. Q等待P继续，直到P等待或退出；
1. 规定signal为管程过程中的最后一个操作，执行完后进程即刻退出管程。

此时要么把挂起进程放入入口等待队列，该进程就会与其他等待进入管程的进程一起竞争。但从某种角度上来说，该挂起进程已经在管程中执行了部分任务，使他优先于其他等待进程是有意义的。因此，我们可以建议一个紧急等待队列，将该挂起进程放入其中。

## 管程模型与信号量模型的区别

使用信号量机制时，互斥和同步都属于进程自己的责任，同步代码(PV操作等)分散在进程各处；各进程还要仔细安排PV操作的顺序，谨慎定义信号量的数值，遗漏一个V操作都有可能时其他进程永远挂起。

而管程构造了自己的互斥机制，由编译器实现，通常使用信号量机制。管程优于信号量之处在于**所有同步机制都被限制在管程内部**，因此易于验证同步的正确性，易于检查出错误。

## 历史上的管程模型

并发编程领域已经发展半个世纪了，管程也经历了多种模型。分别是：
1. Hasen模型；
1. Hoare模型；
1. Mesa模型；

现在广泛应用的是Mesa模型。
![pop](/images/202004/2020年4月3日22点43分1.png "image")

## java中的管程

java中每个对象都可以关联一个管程，通过在Java对象头中的mark word中存储了指向monitor的指针。synchronized关键字、wait()、notify()、notifyAll()这几个方法都是java中管程实现的一部分，其底层是调用了ObjectMonitor的方法。详细内容可参考[此处](https://www.jianshu.com/p/32e1361817f0)

java参考了Mesa模型，语言内置的管程实现(synchronized)对Mesa模型做了简化。Mesa模型中，条件变量可以有多个，Java语言内置的管程里只有一个条件变量(就是对象锁)。如下图所示：
![pop](/images/202004/2020年4月3日23点02分1.png "image")

juc中实现的管程支持多个条件变量，不过juc中的锁，需要开发人员自己进行加锁和解锁操作。

## 总结

并发编程领域的两大问题--互斥、同步，都可以通过管程来解决。管程的解决方法如下：

* 互斥：管程将共享变量及对共享变量的操作统一封装起来,并通过一个互斥锁来保证同一时刻只允许一个进程进入管程，以实现对共享变量的互斥访问。这个互斥锁一般是由编译器自动添加的。
* 同步(时序)：管程中引入了条件变量，而且每个条件变量都对应有一个等待队列，通过这两者来提供同步的机制。条件变量有wait()、signal()两种操作原语，管程中的过程通过调用条件变量的操作原语来封装了进程的同步操作，使进程在条件不满足时被阻塞，条件满足时被唤醒。进程调用管程中的过程就可以实现同步。但管程的过程里何时应该阻塞进程，何时唤醒进程都是由程序员自己决定的，管程模型并不知道你的程序逻辑。

管程和信号量是等价的，所谓等价指的是用管程能够实现信号量，也能用信号量实现管程。但是管程更容易使用，所以 Java 选择了管程。

## References

* [维基百科](https://zh.wikipedia.org/wiki/%E7%9B%A3%E8%A6%96%E5%99%A8_%28%E7%A8%8B%E5%BA%8F%E5%90%8C%E6%AD%A5%E5%8C%96%29)
* [百度百科](https://baike.baidu.com/item/%E7%AE%A1%E7%A8%8B)
* [知乎-应该如何理解管程？](https://www.zhihu.com/question/30641734)
* [\[Java并发-6\]“管程”-java管程初探](https://segmentfault.com/a/1190000019008296)
* [Synchronized之管程](https://www.jianshu.com/p/32e1361817f0)
