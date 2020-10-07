##  CopyOnWriteArrayList与ConcurrentHashMap原理解析

>  郑重声明: 本文首发于[人工博客](https://www.94rg.com)

### 1、CopyOnWriteArrayList

这两个都是非常常用的并发类，先从CopyOnWriteArrayList讲起。这个类我们从名字可以看出，他是在进行写操作时进行复制，因而其它线程进行读操作时不会出现并发问题。它的实现也很简单，我们来看一段简单源码：

```
public boolean add(E e) {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        Object[] elements = getArray();
        int len = elements.length;
        Object[] newElements = Arrays.copyOf(elements, len + 1);
        newElements[len] = e;
        setArray(newElements);
        return true;
    } finally {
        lock.unlock();
    }
}
```

它在进行add操作时先加锁，然后将数组内容复制到一个新数组中，然后在新数组上进行add操作。操作完后再将旧数组的指针指向新数组，解锁。 



```
public E get(int index) {
    return get(getArray(), index);
}
```



get操作则就连锁都没有了，非常简单。



CopyOnWriteArrayList体现了一个非常重要的思想，就是“**读写分离**”，它非常适合读操作频繁，但写操作很少的情况。

优点：CopyOnWriteArrayList适合使用在读操作远远大于写操作的场景里，比如缓存。发生修改时候做copy，新老版本分离，

保证读的高性能，适用于以读为主的情况。

缺点：CopyOnWriteArrayList采用“写入时复制”策略，对容器的写操作将导致的容器中基本数组的复制，性能开销较大。所以在有写操作的情况下，CopyOnWriteArrayList性能不佳，而且如果容器容量较大的话容易造成溢出。


### 2、ConcurrentHashMap

下面再说一说ConcurrentHashMap，它体现了另一个非常重要的思想，那就是分段锁。它比Hashtable，优化的一点就是进行了分段加锁而不是将整个数组都锁上。

我先来介绍一下几种常见的锁优化方案：

#### 2.1、缩小锁范围

优化前

```
public synchronized void synchronizedOnMethod(){ //粗粒度直接在方法上加synchronized,这样会提高锁冲突的概率
           prefix();
           try {
               TimeUnit.SECONDS.sleep(1);
           }catch (InterruptedException e){
           }
           post();
       }
       private void post(){
           try {
               TimeUnit.SECONDS.sleep(1);
           }catch (InterruptedException e){
           }
       }
       private void prefix(){
           try {
               TimeUnit.SECONDS.sleep(1);
           }catch (InterruptedException e){
           }
       }
   }
```



优化后

```
//假设prefix和post方法是线程安全的（与锁无关的代码）
static class SynchronizedClazz{
        public void mineSynOnMethod(){
            prefix();
            synchronized (this){ //synchronized代码块只保护有竞争的代码
                try {
                    TimeUnit.SECONDS.sleep(1);
                }catch (InterruptedException e){
                }
            }
            post();
        }
```



#### 2.2 分离锁

优化前

```
static class DecomposeClazz{
        private final Set<String> allUsers = new HashSet<String>();
        private final Set<String> allComputers = new HashSet<String>();
        
        public synchronized void addUser(String user){ //公用一把锁
            allUsers.add(user);
        }
        
        public synchronized void addComputer(String computer){
            allComputers.add(computer);
        }
    }
```



优化后

```
static class DecompossClazz2{
        private final Set<String> allUsers = new HashSet<String>();
        private final Set<String> allComputers = new HashSet<String>();
        public void addUser(String user){ //分解为两把锁
            synchronized (allUsers){
                allUsers.add(user);
            }
        }
        public void addComputer(String computer){
            synchronized (allComputers){
                allComputers.add(computer);
            }
        }
    }
```

#### 2.3 分段锁

```
package com.app.JavaMaven;
import java.util.HashMap;
import java.util.Map;
 

public class ConcurrentHashMapTest {
 
    @org.junit.Test
    public void test() {
        final MyConcurrentHashMap map = new MyConcurrentHashMap();
 
//        Map map = new HashMap();
 
        new Thread(new Runnable() {
            @Override
            public void run() {
                while (true)
                    map.put("100", "100");
            }
        }).start();
 
        new Thread(new Runnable() {
            @Override
            public void run() {
                while (true)
                    map.put("100", null);
            }
        }).start();
 
        new Thread(new Runnable() {
            @Override
            public void run() {
                while (true)
                    if (map.get("100") == null)
                        System.out.println(map.get("100"));
            }
        }).start();
 
        while (true) ;
 
    }
}
 
class MyConcurrentHashMap<K, V> {
    private final int LOCK_COUNT = 16;
    private final Map<K, V> map;
    private final Object[] locks;
 
    public MyConcurrentHashMap() {
        this.map = new HashMap<K, V>();
        locks = new Object[LOCK_COUNT];
        for (int i = 0; i < LOCK_COUNT; i++) {
            locks[i] = new Object();
        }
    }
 
    private int keyHashCode(K k) {
        return Math.abs(k.hashCode() % LOCK_COUNT);
    }
 
    public V get(K k) {
        int keyHashCode = keyHashCode(k);
        synchronized (locks[keyHashCode % LOCK_COUNT]) {
            return map.get(k);
        }
    }
 
    public V put(K k, V v) {
        int keyHashCode = keyHashCode(k);
        synchronized (locks[keyHashCode % LOCK_COUNT]) {
            return map.put(k, v);
        }
    }
 
}
```

![人工博客](http://oss.94rg.com/oneblog/20200516153233593.jpg-94rg002)  



------

> 版权声明：本文为人工博客的原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接及本声明。
> 本文链接：[https://www.94rg.com/article/1771](https://www.94rg.com/article/1771)