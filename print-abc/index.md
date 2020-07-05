# 顺序多次打印ABC的几种实现方式


<!--more-->

## 基于Semaphore

一个先后关系需要一个信号量来进行控制，n个先后关系就需要n个信号量。(见操作系统书中例子)这个例子是其有向环图版本。
```java
// 设计思想是把一个代表打印资格的信号量在多个线程之间按顺序传递.
public class Test {
    public static void main(String[] args) throws InterruptedException {
        Semaphore a = new Semaphore(1);
        Semaphore b = new Semaphore(0);
        Semaphore c = new Semaphore(0);

        ExecutorService poolService = Executors.newFixedThreadPool(3);

        // 打印次数
        Integer count = 10;

        poolService.execute(new Worker(a, b, "A", count));
        poolService.execute(new Worker(b, c, "B", count));
        poolService.execute(new Worker(c, a, "C", count));

        Thread.sleep(1000);
        poolService.shutdown();

    }

    public static class Worker implements Runnable {
        private Semaphore current;
        private Semaphore next;
        private String key;
        private Integer count;

        public Worker(Semaphore current, Semaphore next, String key, Integer count) {
            this.current = current;
            this.next = next;
            this.key = key;
            this.count = count;
        }

        @Override
        public void run() {
            for (int i = 0; i < count; i++) {
                try {
                    // 获取资源
                    current.acquire();
                    System.out.print(key);
                    // 传递资源
                    next.release();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```

## 基于Synchronized

```java
    // 设计思想是, 用一个lock代表打印资格, 谁获取到谁可以打印; 
    // 并且用一个state来控制打印顺序, 一个线程打印完后, 将state变为下一个状态,
    // 即,state在 a->b->c->a->b->...这样流转;
    // 这个思想跟semaphore差不多的
public class Test {

    private static volatile int currentState = 1;

    public static void main(String[] args) throws InterruptedException {

        ExecutorService poolSerivce = Executors.newFixedThreadPool(3);
        int count = 10;
        Object lock = new Object();

        poolSerivce.execute(new Worker("A", 1, 2, count, lock));
        poolSerivce.execute(new Worker("B", 2, 3, count, lock));
        poolSerivce.execute(new Worker("C", 3, 1, count, lock));

        Thread.sleep(1000);
        poolSerivce.shutdown();
    }

    public static class Worker implements Runnable {
        private String key;
        private int targetState;
        private int nextState;
        private int count;
        private Object lock;

        public Worker(String key, int targetState, int nextState, int count, Object lock) {
            this.key = key;
            this.targetState = targetState;
            this.nextState = nextState;
            this.count = count;
            this.lock = lock;
        }

        @Override
        public void run() {
            for (int i = 0; i < count; i++) {
                synchronized (lock) {
                    // 判断当前该谁打印
                    while (currentState != targetState) {
                        try {
                            lock.wait();
                        } catch (Exception e) {
                            e.printStackTrace();
                        }
                    }
                    System.out.print(key);
                    currentState = nextState;
                    lock.notifyAll();
                }
            }
        }
    }
}
```

## 基于ReentrantLock
设计思想跟 synchronized 一样, 但是有多个等待队列, 提升了竞争效率.
```java
    // 设计思想是, 用一个lock代表打印资格, 谁获取到谁可以打印; 
    // 并且用一个state来控制打印顺序, 一个线程打印完后, 将state变为下一个状态,
    // 即,state在 a->b->c->a->b->...控制当前应打印字符的流转;
public class Test {

    private static volatile Integer currentState = 1;
    private static ReentrantLock lock = new ReentrantLock();

    // 一个condition对应一个等待队列, 一个lock可以绑定多个condition, 即支持多个等待队列.
    // 相比synchronized,有多个等待队列, 通知下一个队列时也有针对性, 即 竞争效率比较高
    // 效率最高的应该还是semaphore
    private static Condition conditionA = lock.newCondition();
    private static Condition conditionB = lock.newCondition();
    private static Condition conditionC = lock.newCondition();

    public static void main(String[] args) throws InterruptedException {

        ExecutorService poolSerivce = Executors.newFixedThreadPool(3);
        Integer count = 10;

        poolSerivce.execute(new Worker("A", 1, 2, count, lock, conditionA, conditionB));
        poolSerivce.execute(new Worker("B", 2, 3, count, lock, conditionB, conditionC));
        poolSerivce.execute(new Worker("C", 3, 1, count, lock, conditionC, conditionA));

        Thread.sleep(1000);
        poolSerivce.shutdown();
    }

    public static class Worker implements Runnable {
        private String key;
        //本线程期待的state
        private int targetState;
        private int nextState;
        private Integer count;
        private Lock lock;
        private Condition current;
        private Condition next;


        public Worker(String key, int targetState, int nextState, Integer count, Lock lock, Condition current, Condition next) {
            this.key = key;
            this.targetState = targetState;
            this.nextState = nextState;
            this.count = count;
            this.lock = lock;
            this.current = current;
            this.next = next;
        }

        @Override
        public void run() {
            this.lock.lock();

            try {
                for (int i = 0; i < count; i++) {
                    // 等待流转到目标状态
                    while (currentState != targetState) {
                        // 放进当前的等待队列中
                        current.await();
                    }
                    System.out.print(key);
                    currentState = nextState;
                    // 通知下一个等待队列
                    next.signal();
                }
            } catch (Exception e) {
                e.printStackTrace();
            } finally {
                this.lock.unlock();
            }
        }
    }
}
```

## References

* [Java多线程---顺序打印ABC打印10次的实现-三种实现](https://www.jianshu.com/p/b036dda3f5c8)
* [java - ReentrantLock和Condition实现生产者-消费者](https://juejin.im/post/5a4edb8d6fb9a01c9f5b35cb)
