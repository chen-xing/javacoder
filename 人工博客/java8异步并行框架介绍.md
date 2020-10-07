### 系统的分类

#### CPU密集型

​	CPU密集型会消耗掉大量的CPU资源，例如需要大量的计算，视频渲染啊，仿真啊之类的。这个时候CPU就卯足了劲在运行，这个时候切换线程，反而浪费了切换的时间，效率不高。

​    就像你的大脑是CPU，你本来就在一本心思地写作业，多线程这时候就是要你写会作业，然后立刻敲一会代码，然后在P个图，然后在看个视频，然后再切换回作业。emmmm，过程中你还需要切换（收起来作业，拿出电脑，打开VS…）那你的作业怕是要写到挂科。。。这个时候不太适合使用多线程，你就该一门心思地写作业~

#### IO密集型

​    涉及到网络、磁盘IO的都是IO密集型，这个时候CPU利用率并不高，这个时候适合使用多线程。

同样以你的大脑为例，IO密集型就是“不烧脑”的工作。例如你需要陪小姐姐或者小哥哥聊天，还需要下载一个VS，还需要看我（黑哥）的博客。这个时候如果使用多线程的话会怎么做？

   咦？小哥哥（小姐姐）给你发消息了，回一下TA，然后呢？TA给你回消息肯定需要时间，这个时候你就可以搜索VS的网站，先下安装包，然后一看，哎呦，TA还没给你回消息，然后看会你黑哥的博客。小哥哥（小姐姐）终于回你了，你回一下TA，接着看我的博客，这就是类似于IO密集型。你可以在不同的“不烧脑”的工作之间切换，来达到更高的效率。而不是小姐姐不回我的信息，我就干等，啥都不干，就等，这个效率可想而知，也许，小姐姐（小哥哥）根本就不会回复你~



### Java8新特性 并行流与串行流 Fork Join



**并行流**就是把一个内容分成多个数据块，并用不同的线程分 别处理每个数据块的流。

Java 8 中将并行进行了优化，我们可以很容易的对数据进行并 行操作。

Stream API 可以声明性地通过 **parallel()** 与 **sequential()** 在并行流与顺序流之间进行切换。

![](https://img2018.cnblogs.com/blog/1467309/201901/1467309-20190126112837479-1891478980.png)





####  **Fork/Join 框架与传统线程池的区别**

采用 “工作窃取”模式（work-stealing）： 当执行新的任务时它可以将其拆分分成更小的任务执行，并将小任务加到线 程队列中，然后再从一个随机线程的队列中偷一个并把它放在自己的队列中。 相对于一般的线程池实现,fork/join框架的优势体现在对其中包含的任务的 处理方式上.在一般的线程池中,如果一个线程正在执行的任务由于某些原因 无法继续运行,那么该线程会处于等待状态.而在fork/join框架实现中,如果 某个子问题由于等待另外一个子问题的完成而无法继续运行.那么处理该子 问题的线程会主动寻找其他尚未运行的子问题来执行.这种方式减少了线程 的等待时间,提高了性能.

 

```
import java.util.concurrent.RecursiveTask;

public class ForkJoinCaculate extends RecursiveTask<Long> {

    private long start;
    private long end;

    public ForkJoinCaculate(long start, long end) {
        this.start = start;
        this.end = end;
    }

    private static final long THRESHOLD = 10000L;

    @Override
    protected Long compute() {
        long length = end - start;

        if(length < THRESHOLD) {
            long sum = 0;
            for (long i = start; i <= end; i++) {
                sum +=i;
            }
            return sum;
        } else {
            long middle = (end + start) / 2; //中间位置
            ForkJoinCaculate left = new ForkJoinCaculate(start,middle);
            left.fork();    //拆分子任务，同时压入线程队列
            ForkJoinCaculate right = new ForkJoinCaculate(middle+1,end);
            right.fork();
            return left.join() + right.join();

        }
    }
}
```



```
import org.junit.Test;

import java.time.Duration;
import java.time.Instant;
import java.util.concurrent.ForkJoinPool;
import java.util.concurrent.ForkJoinTask;
import java.util.stream.LongStream;

public class TestForkJoin {

    public long max = 1000000000L;


    /**
     * 多线程fork Join 方式执行相加
     */
    @Test
    public void test01() {
        Instant start = Instant.now();
        ForkJoinPool pool = new ForkJoinPool();
        ForkJoinTask<Long> task = new ForkJoinCaculate(0,max);
        Long sum = pool.invoke(task);
        System.out.println(sum);
        Instant end = Instant.now();
        System.out.println("耗时：" + Duration.between(start,end));
    }

    /**
     * 单线程普通for循环
     */
    @Test
    public void test02() {
        Instant start = Instant.now();
        long sum = 0L;
        for (long i = 0; i <= max ; i++) {
            sum += i;
        }

        System.out.println(sum);
        Instant end = Instant.now();
        System.out.println("耗费时间为：" + Duration.between(start,end).toMillis());
    }



    @Test
    public void test03() {
        System.out.println("java8 并行流");

        Instant start = Instant.now();

        long sum = LongStream.rangeClosed(0, max)
                .parallel()
                .reduce(Long::sum).getAsLong();

        System.out.println(sum);

        Instant end = Instant.now();

        System.out.println("执行耗时：" + Duration.between(start,end).toMillis());
    }






}
```



