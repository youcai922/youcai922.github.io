#### Spring和SpringMVC的区别

Spring是轻量级的一站式容器框架，SpringMVC是MVC模式的WEB开发框架



#### SpringBoot的优势：

- 快速构建独立的Spring应用
- 内嵌Tomcat，Jetty和Undertow服务器
- 极少的代码生成和XML配置



#### Spring两大核心----------**IOC（控制反转）和AOP（面向切面编程）**

IOC(控制反转)也叫DI(依赖注入)：传统的框架中，如果我们需要使用另外一个类，我们需要new一个这个类的对象，然后再使用；而Spring中我们可以把类交给Spring容器管理，在我们使用的时候Spring就会给我们提供

AOP（面向切面编程）：通过@Aspect声明一个切面类，然后调用一系列方法，实现在切面各个环节调用不同方法的功能。

应用场景：1、日志记录；2、权限验证；3、效率检查；4、事务管理（Spring的事务是AOP实现的）



#### Spring中的IOC

IOC：IOC是一种设计思想，就是 **将原本在程序中手动创建对象的控制权，交由Spring框架来管理。**spring利用工厂模式和反射机制实现，负责创建对象，使用依赖注入（dependency injection，DI）管理它们，将对象集中起来，配置对象，管理对象的整个生命周期。

- 依赖注入方式：注解注入，set注入，构造器注入，静态工厂注入

- ```java
  //注解注入
  @Service
  public class AdminService {
      //code
  }
  ```
  

#### IOC的好处有哪些？

- IOC或依赖注入最小化应用程序代码量。 

- 它使测试应用程序变得容易，因为单元测试中不需要单例或JNDI查找机制。 

- 以最小的代价和最少的干扰来促进松耦合。 

- IOC容器支持快速实例化和懒加载。 



#### Spring中的AOP怎么实现的

**AOP就是基于动态代理的**，如果要代理的对象，实现了某个接口，那么Spring AOP会使用**JDK Proxy**，去创建代理对象，而对于没有实现接口的对象，就无法使用 JDK Proxy 去进行代理了，这时候Spring AOP会使用**Cglib** ，这时候Spring AOP会使用 **Cglib** 生成一个被代理对象的子类来作为代理



#### MVC三层架构

DAO层（data access object）：数据访问层（模型层）
属于一种比较底层，比较基础的操作，具体到对于某个表的增删改查，也就是说某个DAO一定是和数据库的某一张表一一对应的，其中封装了增删改查基本操作，建议DAO只做原子操作，增删改查。

Service层：（View视图层）
Service层叫服务层，被称为服务，粗略的理解就是对一个或多个DAO进行的再次封装，封装成一个服务，所以这里也就不会是一个原子操作了，需要事物控制。

Controler层：（控制层）
Controler负责请求转发，接受页面过来的参数，传给Service处理，接到返回值，再传给页面。

总结：
个人理解DAO面向表，Service面向业务。后端开发时先数据库设计出所有表，然后对每一张表设计出DAO层，然后根据具体的业务逻辑进一步封装DAO层成一个Service层，对外提供成一个服务。

**普通三层架构：界面层（UI），逻辑层（BLL），数据访问层（DAL）**，普通的三层架构适用于所有的应用，而MVC只适用于WEB应用



#### Spring中Bean创建的生命周期有哪些步骤

![深究Spring中Bean的生命周期](https://www.javazhiyin.com/wp-content/uploads/2019/05/java0-1558500658.jpg)

1. Spring启动，查找并加载需要被Spring管理的bean，进行Bean的实例化，如果未加载，调用createBean进行初始化

2. Bean实例化后对将Bean的引入和值注入到Bean的属性中

3. 处理 Aware相关接口接口

   1. 如果Bean实现了BeanNameAware接口的话，Spring将Bean的Id传递给setBeanName()方法
   2. 如果Bean实现了BeanClassLoaderAware接口，调用setBeanClassLoader()方法，传入ClassLoader对象的实例。
   3. 如果Bean实现了BeanFactoryAware接口的话，Spring将调用setBeanFactory()方法，将BeanFactory容器实例传入
   4. 如果Bean实现了ApplicationContextAware接口的话，Spring将调用Bean的setApplicationContext()方法，将bean所在应用上下文引用传入进来。

4. 如果Bean实现了BeanPostProcessor接口，Spring就将调用他们的postProcessBeforeInitialization()方法。

5. 如果Bean 实现了InitializingBean接口，Spring将调用他们的afterPropertiesSet()方法。类似的，如果bean使用init-method声明了初始化方法，该方法也会被调用

6. 如果Bean 实现了BeanPostProcessor接口，Spring就将调用他们的postProcessAfterInitialization()方法。

7. 此时，Bean已经准备就绪，可以被应用程序使用了。他们将一直驻留在应用上下文中，直到应用上下文被销毁。

8. 如果bean实现了DisposableBean接口，Spring将调用它的destory()接口方法，同样，如果bean使用了destory-method 声明销毁方法，该方法也会被调用。

   ![](https://youcai922.github.io/99.src/img/Spring中Bean初始化过程.png)



#### Spring中的Bean时线程安全的么

Spring本身没有针对Bean做线程安全处理，所以

- 如果Bean是无状态的，那么Bean则是线程安全的 
- 如果Bean是有状态的，那么Bean则不是线程安全的

另外，Bean是不是线程安全，跟Bean的作⽤域没有关系，Bean的作⽤域只是表示Bean的⽣命周期范围，对于任何⽣命周期的Bean都是⼀个对象，这个对象是不是线程安全的，还是得看这个Bean对象本身

#### Spring中Bean的作用域

- Singleton：默认作用域，单例Bean，每个容器只有一个Bean的实例
- prototype：为每一个bean请求创建一个实例。
- request：为每一个request请求创建一个实例，在请求完成以后，bean会失效并被垃圾回收器回收。
- session：与request范围类似，同一个session会话共享一个实例，不同会话使用不同的实例。
- global-session：全局作用域，所有会话共享一个实例。如果想要声明让所有会话共享的存储变量的话，那么这全局变量需要存储在global-session中。
  

#### ApplicaitonContext和BeanFactory有什么区别

BeanFactory是Spring中非常核心的组件，表示Bean⼯⼚，可以⽣成Bean，维护Bean，⽽ ApplicationContext继承了BeanFactory，所以ApplicationContext拥有BeanFactory所有的特点，也是⼀个Bean工厂，但是ApplicationContext除开继承了BeanFactory之外，还继承了诸如 EnvironmentCapable、MessageSource、ApplicationEventPublisher等接⼝，从⽽ApplicationContext还有获取系统环境变量、国际化、事件发布等功能，这是BeanFactory所不具备的



#### Spring中事务是如何实现的

1. Spring事务底层是基于数据库事务和AOP机制的
2. ⾸先对于使⽤了@Transactional注解的Bean，Spring会创建⼀个代理对象作为Bean 
3. 当调用代理对象的⽅法时，会先判断该⽅法上是否加了@Transactional注解
4. 如果加了，那么则利用事务管理器创建⼀个数据库连接
5. 并且修改数据库连接的autocommit属性为false，禁⽌此连接的⾃动提交，这是实现Spring事务非常重要的⼀步
6. 然后执⾏当前⽅法，⽅法中会执行sql
7. 执⾏完当前⽅法后，如果没有出现异常就直接提交事务
8. 如果出现了异常，并且这个异常是需要回滚的就会回滚事务，否则仍然提交事务
9. Spring事务的隔离级别对应的就是数据库的隔离级别
10. Spring事务的传播机制是Spring事务⾃⼰实现的，也是Spring事务中最复杂的
11. Spring事务的传播机制是基于数据库连接来做的，⼀个数据库连接⼀个事务，如果传播机制配置为 需要新开⼀个事务，那么实际上就是先建⽴⼀个数据库连接，在此新数据库连接上执⾏sql



#### Spring中什么时候@Transactional会失效

因为Spring事务是基于代理来实现的，所以某个加了@Transactional的⽅法只有是被代理对象调⽤时， 那么这个注解才会⽣效，所以如果是被代理对象来调⽤这个⽅法，那么@Transactional是不会失效的。

同时如果某个⽅法是private的，那么@Transactional也会失效，因为底层cglib是基于⽗⼦类来实现 的，子类是不能重载⽗类的private⽅法的，所以⽆法很好的利⽤代理，也会导致@Transactianal失效



#### Spring容器启动流程是什么样的

- 在创建Spring容器，也就是启动Spring时：
- ⾸先会进⾏扫描，扫描得到所有的BeanDefinition对象，并存在⼀个Map中
- 然后筛选出⾮懒加载的单例BeanDefinition进⾏创建Bean，对于多例Bean不需要在启动过程中去 进⾏创建，对于多例Bean会在每次获取Bean时利⽤BeanDefinition去创建
- 利⽤BeanDefinition创建Bean就是Bean的创建⽣命周期，这期间包括了合并BeanDefinition、推断 构造⽅法、实例化、属性填充、初始化前、初始化、初始化后等步骤，其中AOP就是发⽣在初始化 后这⼀步骤中
- 单例Bean创建完了之后，Spring会发布⼀个容器启动事件
- Spring启动结束
- 在源码中会更复杂，⽐如源码中会提供⼀些模板⽅法，让⼦类来实现，⽐如源码中还涉及到⼀些 BeanFactoryPostProcessor和BeanPostProcessor的注册，Spring的扫描就是通过 BenaFactoryPostProcessor来实现的，依赖注⼊就是通过BeanPostProcessor来实现的
- 在Spring启动过程中还会去处理@Import等注解




#### Spring中的设计模式

![微信截图_20210819145418.png](https://youcai922.github.io/99.src/img/微信截图_20210819145418.png)

- 工厂模式：Spring使用工厂模式，通过BeanFactory和ApplicationContext来创建对象

- 单例模式：Bean默认为单例模式

- 策略模式：例如Resource的实现类，针对不同的资源文件，实现了不同方式的资源获取策略

- 代理模式：Spring的AOP功能用到了JDK的动态代理和CGLIB字节码生成技术

- 模板方法：可以将相同部分的代码放在父类中，而将不同的代码放入不同的子类中，用来解决代码重复的问题。比如RestTemplate, JmsTemplate, JpaTemplate

- 适配器模式：Spring AOP的增强或通知（Advice）使用到了适配器模式，Spring MVC中也是用到了适配器模式适配Controller

- 观察者模式：Spring事件驱动模型就是观察者模式的一个经典应用。

- 桥接模式：可以根据客户的需求能够动态切换不同的数据源。比如我们的项目需要连接多个数据库，客户在每次访问中根据需要会去访问不同的数据库

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



#### SpringBoot是如何启动Tomcat的

- 首先，SpringBoot在启动的时候会创建一个Spring容器

- 在创建Spring容器过程中，会利用@ConditionOnClass技术来判断当前classpath中是否存在Tomcat依赖，如果存在则会生成一启动Tomcat的Bean

- Spring容器创建完之后，就会获取启动Tomcat的Bean，并创建Tomcat对象，并绑定端口等，然后启动Tomcat



#### Spring中配置文件加载顺序

优先级从高到低，⾼优先级的配置覆盖低优先级的配置，所有配置会形成互补配置。

1. 命令⾏参数。所有的配置都可以在命令行上进⾏指定
2. Java系统属性（System.getProperties()）
3. 操作系统环境变量
4. jar包外部的application-{profile}.properties或application.yml(带spring.profile)配置⽂件
5. jar包内部的application-{profile}.properties或application.yml(带spring.profile)配置⽂件 再来加 载不带profile
6. jar包外部的application.properties或application.yml(不带spring.profile)配置⽂件
7. jar包内部的application.properties或application.yml(不带spring.profile)配置⽂件
8. @Configuration注解类上的@PropertySource



#### @Autowire和@Resourese注解的区别

@Autowired按类型，是Spring的。@Resource按名字，是JDK的，



