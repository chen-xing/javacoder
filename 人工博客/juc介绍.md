# juc介绍

## 1、日常工作中的并发

今天的日程

+ jira待处理的任务两个
+ 完成测试环境的发布，通知测试开始验证
+ 钉钉联系客户了解问题发生的过程
+ 迭代新功能开发



**实际过程中我们怎么做？**

1. 先在发布平台上把发布的过程触发起来；
2. 钉钉给客户发消息询问问题详情；
3. 打开jira,开始分析问题
4. 过一段时间回头看下发布的过程是否已经完成了，客户是否有回复



>  并发充分利用cpu资源，提高程序的响应速度

## 2、并发常见的问题

**线程安全**  

当多个线程在共享同一个变量，做读写的时候，会由于其他线程的干扰，导致数据误差，就会出现线程安全问题

**解决办法**

+ 锁
+ cas

## 3、juc是什么

>  java.util.concurrent  java 并发工具包



## 4、锁

+ Synchronized
+ Lock   ReentrantLock
+ ReadWriteLock
+ cas

 **Synchronized 和 Lock 区别**
 1、Synchronized 内置的Java关键字， Lock 是一个Java类
 2、Synchronized 无法判断获取锁的状态，Lock 可以判断是否获取到了锁
 3、Synchronized 会自动释放锁，lock 必须要手动释放锁！如果不释放锁，死锁
 4、Synchronized 线程 1（获得锁，阻塞）、线程2（等待，傻傻的等）；Lock锁就不一定会等待下 去；
 5、Synchronized 可重入锁，不可以中断的，非公平；Lock ，可重入锁，可以 判断锁，非公平（可以 自己设置）；
 6、Synchronized 适合锁少量的代码同步问题，Lock 适合锁大量的同步代码！

## 5、juc并发工具类介绍

### 5.1、ThreadPoolExecutor

**线程池的好处:**

1、降低资源的消耗

2、提高响应的速度

3、方便管理。



**线程复用、可以控制最大并发数、管理线程**



```
 public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              RejectedExecutionHandler handler) {
        this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue,
             Executors.defaultThreadFactory(), handler);
    }
```



![image-20210715204812166](https://oss.94rg.com/figure_bed/20210716183002.png-94rg002)



### 5.2、并发协同

| CountDownLatch    | 线程计数器（ 减法）countDown |
| ----------------- | ---------------------------- |
| CyclicBarrier     | 线程计数器（加法） await()   |
| Semaphore         | 信号量 限流                  |
| ForkJoin          | 单机版的 map reduce          |
| CompletableFuture | 异步回调                     |





| CountDownLatch | CyclicBarrier |
| -------------- | ------------- |
|    减计数方式            |   加计数方式            |
|        计算为0时释放所有等待的线程        |      计数达到指定值时释放所有等待线程         |
| 计数为0时，无法重置 | 计数达到指定值时，计数置为0重新开始 |
| 调用countDown()方法计数减一，调用await()方法只进行阻塞，对计数没任何影响 | 调用await()方法计数加1，若加1后的值不等于构造方法的值，则线程阻塞 |
| 不可重复利用 | 可重复利用 |

**CountDownLatch** : **一个线程**(或者多个)， 等待另外**N个线程**完成**某个事情**之后才能执行。  **CyclicBarrier**     : **N个线程**相互等待，任何一个线程完成之前，所有的线程都必须等待。
这样应该就清楚一点了，对于CountDownLatch来说，重点是那个**“一个线程”**, 是它在等待， 而另外那N的线程在把**“某个事情”**做完之后可以继续等待，可以终止。而对于CyclicBarrier来说，重点是那**N个线程**，他们之间任何一个没有完成，所有的线程都必须等待。



**countdownlatch**

```
 public void countDownlatchTest() throws InterruptedException {
        CountDownLatch countDownLatch = new CountDownLatch(5);
        for (int i = 0; i < 5; i++) {
            final Integer count = i + 1;
            new Thread(
                            () -> {
                                try {
                                    TimeUnit.SECONDS.sleep(count);
                                } catch (InterruptedException e) {
                                    e.printStackTrace();
                                }
                                System.out.println("当前循环第" + String.valueOf(count) + "次");
                                countDownLatch.countDown();
                            })
                    .start();
        }
        countDownLatch.await(10, TimeUnit.SECONDS);
        System.out.println("over....");
    }
```



**CyclicBarrier**

```
public void cyclicBarrierTest() throws InterruptedException {

        CyclicBarrier cyclicBarrier =
                new CyclicBarrier(
                        2,
                        () -> {
                            System.out.println("组队成功");
                        });
        for (int i = 0; i < 6; i++) {
            final Integer count = i + 1;
            new Thread(
                    () -> {
                        try {
                            try {
                                cyclicBarrier.await();
                            } catch (BrokenBarrierException e) {
                                e.printStackTrace();
                            }
                            doSomething(Thread.currentThread().getName());
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                    })
                    .start();
        }
        while (Thread.activeCount()>2){
            System.out.println("当前的线程数是"+Thread.activeCount());
            TimeUnit.SECONDS.sleep(1);
        }
        System.out.println("over....");
    }

    private void doSomething(String name) throws InterruptedException {
        System.out.println(name+"开始准备");
        TimeUnit.SECONDS.sleep(3);
        System.out.println(name+"开始到达终点");
    }
```



**fork join**

```
public class ForkJoinPoolDemo {
    private static final Integer MAX = 200;

    static class MyForkJoinTask extends RecursiveTask<Integer> {
        // 子任务开始计算的值
        private Integer startValue;

        // 子任务结束计算的值
        private Integer endValue;

        public MyForkJoinTask(Integer startValue, Integer endValue) {
            this.startValue = startValue;
            this.endValue = endValue;
        }

        @Override
        protected Integer compute() {
            // 如果条件成立，说明这个任务所需要计算的数值分为足够小了
            // 可以正式进行累加计算了
            if (endValue - startValue < MAX) {
                System.out.println(
                        "开始计算的部分：startValue = " + startValue + ";endValue = " + endValue);
                Integer totalValue = 0;
                for (int index = this.startValue; index <= this.endValue; index++) {
                    totalValue += index;
                }
                return totalValue;
            }
            // 否则再进行任务拆分，拆分成两个任务
            else {
                MyForkJoinTask subTask1 =
                        new MyForkJoinTask(startValue, (startValue + endValue) / 2);
                subTask1.fork();
                MyForkJoinTask subTask2 =
                        new MyForkJoinTask((startValue + endValue) / 2 + 1, endValue);
                subTask2.fork();
                return subTask1.join() + subTask2.join();
            }
        }
    }

    public static void main(String[] args) {
        // 这是Fork/Join框架的线程池
        ForkJoinPool pool = new ForkJoinPool();
        ForkJoinTask<Integer> taskFuture = pool.submit(new MyForkJoinTask(1, 1001));
        try {
            Integer result = taskFuture.get();
            System.out.println("result = " + result);
        } catch (InterruptedException | ExecutionException e) {
            e.printStackTrace(System.out);
        }
    }
```





## 6、juc基石

+ 可见性：可见性是指当多个线程访问同一个变量时，一个线程修改了这个变量的值，其他线程能够立即看得到修改的值。
+ 原子性：即一个操作或者多个操作 要么全部执行并且执行的过程不会被任何因素打断，要么就都不执行
+ 有序性：即程序执行的顺序按照代码的先后顺序执行。



**要想并发程序正确地执行，必须要保证原子性、可见性以及有序性。只要有一个没有被保证，就有可能会导致程序运行不正确。**



**可见性**

```
private static volatile  int num = 0;

private static void volatileTest() throws InterruptedException {
        new Thread(
                        () -> {
                            while (num == 0) {
                            }
                        })
                .start();

        TimeUnit.SECONDS.sleep(1);
        num = 1;
        System.out.println(num);
    }
```

JMM关于synchronized的两条规定:

（1）线程解锁前，必须把共享变量的最新值刷新到主内存中

（2）线程加锁时，将清空工作内存中共享变量的值，从而使用共享变量时需要从主内存中重新获取最新的值

（注意：加锁与解锁需要是同一把锁）

synchronized具体过程是：

获得同步锁；
清空工作内存；
从主内存拷贝对象副本到工作内存；
执行代码(计算或者输出等)；
刷新主内存数据；
释放同步锁。



各种耗时但cpu占用不高的的操作，都会达到内存同步的效果

+ 查询数据库
+ 发送http
+ new 大对象
+ 读写文件
+ 

JVM针对现在的硬件水平已经做了很大程度的**优化，基本上很大程度的保障了工作内存和主内存的及时同步**，相当于默认使用了volitale。但只是最大程度。在**CPU资源一直被占用的时候**，工作内存与主内存中间的同步，也就是变量的可见性就不会那么及时！



**原子性**

```
private static volatile  int num = 0;
//模拟10个线程去操作
        for (int i=0;i<10;i++){
            new Thread(()->{
                for(int j=1;j<=1000;j++){
                    num++;
                }
            }).start();
        }

        while (Thread.activeCount()>2){
            System.out.println("当前的线程数是"+Thread.activeCount()+",结果是:"+num);
            TimeUnit.SECONDS.sleep(1);
        }
        System.out.println("最终的结果是："+num);
```






+ atomic  使用原子类保证原子性
+ Unsafe
	+  底层操作系统级别的保障
	+  自旋锁重试循环解除
+ AtomicReference (支持多对象需要保持原子性)
+ AtomicStampedReference（解决ABA问题）



![atomic系列](https://cdn.jsdelivr.net/gh/chen-xing/figure_bed_02/cdn/20210803160142608.png)



![cas不会死循环](https://cdn.jsdelivr.net/gh/chen-xing/figure_bed_02/cdn/20210803161829568.png)



![image-20210811094956275](https://oss.94rg.com/figure_bed/20210811095005.png-94rg002)



## 7、参考

+ https://blog.csdn.net/ls5718/article/details/52563959
+ https://www.cnblogs.com/dolphin0520/p/3920373.html
+ https://www.infoq.cn/article/fork-join-introduction
+ https://www.cnblogs.com/xiaorenwu702/p/3977833.html

