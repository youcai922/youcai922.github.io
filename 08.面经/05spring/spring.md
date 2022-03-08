#### Spring和SpringMVC的区别

Spring是轻量级的一站式容器框架，SpringMVC是MVC模式的WEB开发框架



#### Spring两大核心

IOC（控制反转）和AOP（面向切面编程）

IOC(控制反转)也叫DI(依赖注入)：传统的框架中，如果我们需要使用另外一个类，我们需要new一个这个类的对象，然后再使用；而Spring中我们可以把类交给Spring容器管理，在我们使用的时候Spring就会给我们提供

AOP（面向切面编程）：通过@Aspect声明一个切面类，然后调用一系列方法，实现在切面各个环节调用不同方法的功能。

应用场景：1、日志记录；2、权限验证；3、效率检查；4、事务管理（Spring的事务是AOP实现的）



#### MVC三层架构

DAO层（data access object）：数据访问层
属于一种比较底层，比较基础的操作，具体到对于某个表的增删改查，也就是说某个DAO一定是和数据库的某一张表一一对应的，其中封装了增删改查基本操作，建议DAO只做原子操作，增删改查。

Service层：
Service层叫服务层，被称为服务，粗略的理解就是对一个或多个DAO进行的再次封装，封装成一个服务，所以这里也就不会是一个原子操作了，需要事物控制。

Controler层：
Controler负责请求转发，接受页面过来的参数，传给Service处理，接到返回值，再传给页面。

总结：
个人理解DAO面向表，Service面向业务。后端开发时先数据库设计出所有表，然后对每一张表设计出DAO层，然后根据具体的业务逻辑进一步封装DAO层成一个Service层，对外提供成一个服务。



#### Spring中Bean创建的生命周期有哪些步骤

![深究Spring中Bean的生命周期](https://www.javazhiyin.com/wp-content/uploads/2019/05/java0-1558500658.jpg)

1. Spring启动，查找并加载需要被Spring管理的bean，进行Bean的实例化
2. Bean实例化后对将Bean的引入和值注入到Bean的属性中
3. 如果Bean实现了BeanNameAware接口的话，Spring将Bean的Id传递给setBeanName()方法
4. 如果Bean实现了BeanFactoryAware接口的话，Spring将调用setBeanFactory()方法，将BeanFactory容器实例传入
5. 如果Bean实现了ApplicationContextAware接口的话，Spring将调用Bean的setApplicationContext()方法，将bean所在应用上下文引用传入进来。
6. 如果Bean实现了BeanPostProcessor接口，Spring就将调用他们的postProcessBeforeInitialization()方法。
7. 如果Bean 实现了InitializingBean接口，Spring将调用他们的afterPropertiesSet()方法。类似的，如果bean使用init-method声明了初始化方法，该方法也会被调用
8. 如果Bean 实现了BeanPostProcessor接口，Spring就将调用他们的postProcessAfterInitialization()方法。
9. 此时，Bean已经准备就绪，可以被应用程序使用了。他们将一直驻留在应用上下文中，直到应用上下文被销毁。
10. 如果bean实现了DisposableBean接口，Spring将调用它的destory()接口方法，同样，如果bean使用了destory-method 声明销毁方法，该方法也会被调用。



#### Spring中的Bean时线程安全的么

Spring本身没有针对Bean做线程安全处理，所以

- 如果Bean是无状态的，那么Bean则是线程安全的 
- 如果Bean是有状态的，那么Bean则不是线程安全的

另外，Bean是不是线程安全，跟Bean的作⽤域没有关系，Bean的作⽤域只是表示Bean的⽣命周期范围，对于任何⽣命周期的Bean都是⼀个对象，这个对象是不是线程安全的，还是得看这个Bean对象本身



#### ApplicaitonContext和BeanFactory有什么区别

BeanFactory是Spring中⾮常核⼼的组件，表示Bean⼯⼚，可以⽣成Bean，维护Bean，⽽ ApplicationContext继承了BeanFactory，所以ApplicationContext拥有BeanFactory所有的特点，也是⼀个Bean⼯⼚，但是ApplicationContext除开继承了BeanFactory之外，还继承了诸如 EnvironmentCapable、MessageSource、ApplicationEventPublisher等接⼝，从⽽ApplicationContext还有获取系统环境变量、国际化、事件发布等功能，这是BeanFactory所不具备的



#### Spring中事务是如何实现的

1. Spring事务底层是基于数据库事务和AOP机制的
2. ⾸先对于使⽤了@Transactional注解的Bean，Spring会创建⼀个代理对象作为Bean 
3. 当调⽤代理对象的⽅法时，会先判断该⽅法上是否加了@Transactional注解
4. 如果加了，那么则利⽤事务管理器创建⼀个数据库连接
5. 并且修改数据库连接的autocommit属性为false，禁⽌此连接的⾃动提交，这是实现Spring事务⾮ 常重要的⼀步
6. 然后执⾏当前⽅法，⽅法中会执⾏sql
7. 执⾏完当前⽅法后，如果没有出现异常就直接提交事务
8. 如果出现了异常，并且这个异常是需要回滚的就会回滚事务，否则仍然提交事务
9. Spring事务的隔离级别对应的就是数据库的隔离级别
10. Spring事务的传播机制是Spring事务⾃⼰实现的，也是Spring事务中最复杂的
11. Spring事务的传播机制是基于数据库连接来做的，⼀个数据库连接⼀个事务，如果传播机制配置为 需要新开⼀个事务，那么实际上就是先建⽴⼀个数据库连接，在此新数据库连接上执⾏sql



#### Spring中什么时候@Transactional会失效

因为Spring事务是基于代理来实现的，所以某个加了@Transactional的⽅法只有是被代理对象调⽤时， 那么这个注解才会⽣效，所以如果是被代理对象来调⽤这个⽅法，那么@Transactional是不会失效的。

同时如果某个⽅法是private的，那么@Transactional也会失效，因为底层cglib是基于⽗⼦类来实现 的，⼦类是不能重载⽗类的private⽅法的，所以⽆法很好的利⽤代理，也会导致@Transactianal失效



#### Spring容器启动流程是什么样的

1.在创建Spring容器，也就是启动Spring时：

2. ⾸先会进⾏扫描，扫描得到所有的BeanDefinition对象，并存在⼀个Map中
3. 然后筛选出⾮懒加载的单例BeanDefinition进⾏创建Bean，对于多例Bean不需要在启动过程中去 进⾏创建，对于多例Bean会在每次获取Bean时利⽤BeanDefinition去创建
4. 利⽤BeanDefinition创建Bean就是Bean的创建⽣命周期，这期间包括了合并BeanDefinition、推断 构造⽅法、实例化、属性填充、初始化前、初始化、初始化后等步骤，其中AOP就是发⽣在初始化 后这⼀步骤中
5. 单例Bean创建完了之后，Spring会发布⼀个容器启动事件
6. Spring启动结束
7. 在源码中会更复杂，⽐如源码中会提供⼀些模板⽅法，让⼦类来实现，⽐如源码中还涉及到⼀些 BeanFactoryPostProcessor和BeanPostProcessor的注册，Spring的扫描就是通过 BenaFactoryPostProcessor来实现的，依赖注⼊就是通过BeanPostProcessor来实现的
8. 在Spring启动过程中还会去处理@Import等注解


#### Spring中的设计模式

![微信截图_20210819145418.png](https://youcai922.github.io/99.src/img/微信截图_20210819145418.png)



#### SpringMVC的底层工作原理

1. ⽤户发送请求⾄前端控制器DispatcherServlet
2. DispatcherServlet 收到请求调⽤ HandlerMapping 处理器映射器
3. 处理器映射器找到具体的处理器(可以根据 xml 配置、注解进⾏查找)，⽣成处理器及处理器拦截器 (如果有则⽣成)⼀并返回给 DispatcherServlet
4. DispatcherServlet 调⽤ HandlerAdapter 处理器适配器
5. HandlerAdapter 经过适配调⽤具体的处理器(Controller，也叫后端控制器)
6. Controller 执⾏完成返回 ModelAndView
7. HandlerAdapter 将 controller 执⾏结果 ModelAndView 返回给 DispatcherServlet
8. DispatcherServlet 将 ModelAndView 传给 ViewReslover 视图解析器。
9. ViewReslover 解析后返回具体 View。
10. DispatcherServlet 根据 View 进⾏渲染视图（即将模型数据填充⾄视图中）。
11. DispatcherServlet 响应⽤户。



#### SpringBoot中常用注解及其底层实现

1. @SpringBootApplication注解：这个注解标识了⼀个SpringBoot⼯程，它实际上是另外三个注解 的组合，这三个注解是：

   a. @SpringBootConfiguration：这个注解实际就是⼀个@Configuration，表示启动类也是⼀个 配置类

   b.@EnableAutoConfiguration：向Spring容器中导⼊了⼀个Selector，⽤来加载ClassPath下 SpringFactories中所定义的⾃动配置类，将这些⾃动加载为配置Bean

   c.@ComponentScan：标识扫描路径，因为默认是没有配置实际扫描路径，所以SpringBoot扫 描的路径是启动类所在的当前⽬录

2. @Bean注解：⽤来定义Bean，类似于XML中的标签，Spring在启动时，会对加了@Bean注 解的⽅法进⾏解析，将⽅法的名字做为beanName，并通过执⾏⽅法得到bean对象

3. @Controller、@Service、@ResponseBody、@Autowired都可以说



#### SpringBoot是如何启动Tomcat的

1.首先，SpringBoot在启动的时候会创建一个Spring容器

2.在创建Spring容器过程中，会利用@ConditionOnClass技术来判断当前classpath中是否存在Tomcat依赖，如果存在则会生成一启动Tomcat的Bean

3.Spring容器创建完之后，就会获取启动Tomcat的Bean，并创建Tomcat对象，并绑定端口等，然后启动Tomcat



#### Spring中配置文件加载顺序

优先级从⾼到低，⾼优先级的配置覆盖低优先级的配置，所有配置会形成互补配置。

1. 命令⾏参数。所有的配置都可以在命令⾏上进⾏指定
2. Java系统属性（System.getProperties()）
3. 操作系统环境变量
4. jar包外部的application-{profile}.properties或application.yml(带spring.profile)配置⽂件
5. jar包内部的application-{profile}.properties或application.yml(带spring.profile)配置⽂件 再来加 载不带profile
6. jar包外部的application.properties或application.yml(不带spring.profile)配置⽂件
7. jar包内部的application.properties或application.yml(不带spring.profile)配置⽂件
8. @Configuration注解类上的@PropertySource

