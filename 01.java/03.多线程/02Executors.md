## Executor框架

Executor是用来管理线程池的框架，大体上分为三个部分

- 任务。也就是工作单元，包括被执行任务需要实现的接口：Runnable接口或者Callable接口；
- 任务的执行。也就是把任务分派给多个线程的执行机制，包括Executor接口及继承自Executor接口的ExecutorService接口。
- 异步计算的结果。包括Future接口及实现了Future接口的FutureTask类。



-------------

### Java通过Executor提供四种线程池

- newCachedThreadPool创建一个可缓存线程池，如果线程池长度超过处理需要，可灵活回收空闲线程，若无可回收，则新建线程
- newFixedThreadPool 创建一个定长线程池，可控制线程最大并发数，超出的线程会在队列中等待。
- newScheduledThreadPool 创建一个定长线程池，支持定时及周期性任务执行。
- newSingleThreadExecutor 创建一个单线程化的线程池，它只会用唯一的工作线程来执行任务，保证所有任务按照指定顺序(FIFO, LIFO, 优先级)执行。

### 使用

定义一个线程任务

```java
class ThreadPools implements Runnable{
    private Integer index;
    private ThreadPools(Integer index){
        this.index=index;
    }

    @Override
    public void run() {
        //.....业务处理
        try{
            System.out.println("线程任务处理");
            Thread.sleep(100*index);
            System.out.println("线程标识"+this.toString());
        }catch (InterruptedException e){
            e.printStackTrace();
        }
    }
}
```

- **方案一：newCachedThreadPool**：			创建一个可缓存线程池，应用中存在的线程数可以无限大

```java
public static void main(String[] args){
    ExecutorService newCachedThreadPool = Executors.newCachedThreadPool();
    System.out.println("创建newCachedThreadPool处理任务");
    for (int i=0;i<4;i++){
        final int index=1;
        newCachedThreadPool.execute(new ThreadPools(index));
    }
}
```

- **方案二：newFixedThreadPool**：			  创建一个定长线程池，可控制线程最大并发数，超出的线程会在队列中等待

```java
public static void main(String[] args){
    //设定定长线程数量为2，当达到线程2时，进行等待，只有在有线程处理完之后，才会创建新的线程
    ExecutorService newFixedThreadPool = Executors.newFixedThreadPool(2);
    System.out.println("创建newCachedThreadPool处理任务");
    for (int i=0;i<4;i++){
        final int index=1;
        newFixedThreadPool.execute(new ThreadPools(index));
    }
}
```

- **方案三：newScheduledThreadPool**：		  创建一个定长线程池，支持定时及周期性任务执行。

```java
public static void main(String[] args){
    ScheduledExecutorService scheduledExecutorService = Executors.newScheduledThreadPool(2);
    System.out.println("创建newCachedThreadPool处理任务");
    for (int i=0;i<4;i++){
        final int index=1;
        //创建线程延迟三秒钟执行run方法
        scheduledExecutorService.schedule(new ThreadPools(index),3, TimeUnit.SECONDS);
    }
}
```

- **方案四：newSingleThreadExecutor**：
  创建一个单线程化的线程池，它只会用唯一的工作线程来执行任务，保证所有任务按照指定顺序(FIFO, LIFO, 优先级)执行。

```java
public static void main(String[] args){
    ExecutorService newSingleThreadExecutor = Executors.newSingleThreadExecutor();
    System.out.println("创建newCachedThreadPool处理任务");
    for (int i=0;i<4;i++){
        final int index=1;
        newSingleThreadExecutor.execute(new ThreadPools(index));
    }
}
```



--------------

### FutureTask

