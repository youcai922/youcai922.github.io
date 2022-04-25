### 使用线程池

--------

定义线程池

```java
@EnableAsync
@Configuration
class TaskPoolConfig {
	@Bean("taskExecutor")
	public Executor taskExecutor() {
		ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        //核心线程数
        executor.setCorePoolSize(10);
        //最大线程数
        executor.setMaxPoolSize(20);
        //空闲队列
        executor.setQueueCapacity(200);
        //允许线程空闲时间
        executor.setKeepAliveSeconds(60);
        //线程池名称
        executor.setThreadNamePrefix("taskExecutor-");
        //拒绝策略
        executor.setRejectedExecutionHandler(new ThreadPoolExecutor.CallerRunsPolicy());
       //设置线程池关闭的时候等待所有任务都完成再继续销毁其他的Bean，这样这些异步任务的销毁就会先于Redis线程池的销毁。同时，这里还设置了setAwaitTerminationSeconds(60)，该方法用来设置线程池中任务的等待时间，如果超过这个时候还没有销毁就强制销毁，以确保应用最后能够被关闭，而不是阻塞住。
        executor.setWaitForTasksToCompleteOnShutdown(true);
        executor.setAwaitTerminationSeconds(60);
        return executor;
    }
}
```

#### 拒绝策略

- AbortPolicy：线程池的默认拒绝策略为AbortPolicy，即丢弃任务并抛出RejectedExecutionException异常
- DiscardPolicy：丢弃任务，但是不抛出异常。如果线程队列已满，则后续提交的任务都会被丢弃，且是静默丢弃
- DiscardOldestPolicy：丢弃队列最前面的任务，然后重新提交被拒绝的任务。喜新厌旧的拒绝策略。是否要采用此种拒绝策略，还得根据实际业务是否允许丢弃老任务来认真衡量。
- CallerRunsPolicy：任务被拒绝了，则由调用线程（提交任务的线程）直接执行此任务

