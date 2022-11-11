#### SpringAOP

AOP（面向切面编程）是一种思想，并不是Spring独有的，而是Spring提供了相关的框架

##### 基础概念

- Aspect（切面）：声明切面，类似于类定义，每个Aspect可以包含多个PointCut和Advice
- PoinCut（切点）：用来筛选我们需要在那些地方进行增强逻辑
- Joinpoint（连接点）：通过切点筛选出来要进行增强逻辑的地方
- Advince（通知）：我们对筛选出来的连接点坑定要进行额外的操作，这些额外的操作就是Adivince
  - @Before前置通知：在一个方法执行之前，执行通知。
  - @After后置通知：在一个方法执行之后，不考虑其结果，执行通知。
  - @AfterReturning返回后通知：在一个方法执行之后，只有在方法成功完成时，才能执行通知。
  - @AfterThrowing抛出异常后通知：在一个方法执行之后，只有在方法退出抛出异常时，才能执行通知。
  - @Around环绕通知：在建议方法调用之前和之后，执行通知。
- Target Object（目标对象）：被织入Advince的对象
- Weaving（织入）：将Advince织入到通过Pointcut筛选出来的JoinPoint上，就是JoinPoint执行的时候要执行Advice的逻辑

##### 应用场景：

- Spring中的应用场景：
  - Spring事务（Transactions）
  - Spring缓存（Caching）
  - Spring本地调度（Scheduling）
- 在实际中的应用：
  - 记录日志
  - 权限控制
  - 分布式事务

```java
@Aspect
public class TransactionAspect{
	@Before("execution(* com.test.service.impl.UserServiceImpl.*(..))")
	public void before(){
        System.out.printf("前置增强，比如创建连接对象。。。");
    }
    @After("execution(* com.test.service.impl.UserServiceImpl.*(..))")
    public void after(){
        System.out.printf("后置增强，比如关闭连接对象。。。");
    }
    @AfterRetruning("execution(* com.test.service.impl.UserServiceImpl.*(..))")
    public void afterReturning(){
        System.out.printf("返回通知，比如提交事务。。。");
    }
    @AfterThrowing("execution(* com.test.service.impl.UserServiceImpl.*(..))")
    public void AfterThrowing(){
        System.out.printf("异常通知，比如回滚事务。。。");
    }
    
    //声明一个切面
    @PointCut("execution(public * com.sitech.predeal.workflow.util.hcai.client.Client.LogAndMap(..))")
    public void index_log(){}
    
    //在切面执行后进行一个后置增强。
    @After("index_log()")
    public void doBefore(JoinPoint joinPoint){
        //日志处理，通过joinPoint.getArgs()获取参数数组
        String log_plateform = joinPoint.getArgs()[0].toString();
        String log_transcode = joinPoint.getArgs()[1].toString();
    }
}

//com.sitech.predeal.workflow.util.hcai.client.Client.LogAndMap
public Map LogAndMap(String log_plateform, String log_transcode, String log_request, String log_response, String service_code, String is_syn){
    try {
        log_response = log_response.replaceAll("OK", "0");
        log_response = log_response.replaceAll("ResultCode=0000", "returnCode=0");
        String changResponse = log_response;//定义一个变量，将接口返回的信息进行赋值
        //将接口返回的信息转换成map
        Map map = CommonService.putMap(changResponse);
        return map;
    } catch (Exception e) {
        log.error("errMesage", e);
        return null;
    }
}
```





```java
// 获取目标方法上的注解
private <T extends Annotation> T getMethodAnnotation(ProceedingJoinPoint joinPoint, Class<T> clazz) {
    MethodSignature methodSignature = (MethodSignature) joinPoint.getSignature();
    Method method = methodSignature.getMethod();
    return method.getAnnotation(clazz);
}
```



私有方法无法进行AOP操作

静态方法无法进行AOP操作

请确保切面方法的类被Spring管理
