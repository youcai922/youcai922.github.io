#### SpringBoot中常用注解及其底层实现

1. @SpringBootApplication注解：这个注解标识了⼀个SpringBoot⼯程，它实际上是另外三个注解 的组合，这三个注解是：

   a. @SpringBootConfiguration：这个注解实际就是⼀个@Configuration，表示启动类也是⼀个 配置类

   b.@EnableAutoConfiguration：向Spring容器中导⼊了⼀个Selector，⽤来加载ClassPath下 SpringFactories中所定义的⾃动配置类，将这些⾃动加载为配置Bean

   c.@ComponentScan：标识扫描路径，因为默认是没有配置实际扫描路径，所以SpringBoot扫 描的路径是启动类所在的当前⽬录

2. @Bean注解：⽤来定义Bean，类似于XML中的标签，Spring在启动时，会对加了@Bean注 解的⽅法进⾏解析，将⽅法的名字做为beanName，并通过执⾏⽅法得到bean对象

3. @Controller（控制器层）、@Service（业务层）、@Component（组件）、@Respository（数据访问组件DAO）、@ResponseBody、@Autowired（按照类型自动注入）、@Resource（按照名称进行注入）

4. @Async异步线程，需要添加ThreadPoolConfig类并使用@EnableAsync注解开启

5. @Transaction事务注解



#### @Transaction事务失效

- 未启动spring事务管理功能，需要使用@EnableTransactionManagement来开启事务注解

- 方法非public， @Transaction 可以用在类上、接口上、public方法上，如果将@Trasaction用在了非public方法上，事务将无效

- 数据源未配置事务管理器

  ```java
  @Bean
  public PlatformTransactionManager transactionManager(DataSource dataSource) {
      return new DataSourceTransactionManager(dataSource);
  }
  ```

- 自身调用问题（同一个类中方法调用，可能会导致失效）

  比如有一个类Test，它的一个方法A，A再调用本类的方法B（不论方法B是用public还是private修饰），但方法A没有声明注解事务，而B方法有。则外部调用方法A之后，方法B的事务是不会起作用的。这也是经常犯错误的一个地方。

  ```java
  //m1中通过this的方式调用了m2方法，而this并不是代理对象，this.m2()不会被事务拦截器，所以事务是无效的
  @Component
  public class UserService {
      public void m1(){
          this.m2();
      }
      
      @Transactional
      public void m2(){
          //执行db操作
      }
  }
  //如果外部直接调用通过UserService这个bean来调用m2方法，事务是有效的，上面代码可以做一下调整，如下，@1在UserService中注入了自己，此时m1中的m2事务是生效的
  @Component
  public class UserService {
      @Autowired //@1
      private UserService userService;
   
      public void m1() {
          this.userService.m2();
      }
   
      @Transactional
      public void m2() {
          //执行db操作
      }
  }
  ```

- 异常类型处理：spring只有在感知到异常的时候才进行回滚处理，如果异常被捕获并且处理了，则不会回滚

  ```java
  @Transactional
  public void m1(){
      事务操作1
      try{
          事务操作2，内部抛出了异常
      }catch(Exception e){
          
      }
  }
  ```

- 非同一线程不会回滚

  ```java
  @Transactional
  public void m1() {
      new Thread() {
          一系列事务操作
      }.start();
  }
  ```

- 数据库引擎：常用的MySQL数据库默认使用支持事务的innodb引擎。一旦数据库引擎切换成不支持事务的myisam，那事务就从根本上失效了。







#### springboot获取入参

https://blog.csdn.net/weixin_51128948/article/details/115894071