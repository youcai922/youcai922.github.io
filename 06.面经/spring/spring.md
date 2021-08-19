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
4. 利⽤BeanDefinition创建Bean就是Bean的创建⽣命周期，这期间包括了合并BeanDefinition、推断 构造⽅法、实例化、属性填充、初始化前、初始化、初始化后等步骤，其中AOP就是发⽣在初始化 后这⼀步骤中 5. 单例Bean创建完了之后，Spring会发布⼀个容器启动事件 6. Spring启动结束 7. 在源码中会更复杂，⽐如源码中会提供⼀些模板⽅法，让⼦类来实现，⽐如源码中还涉及到⼀些 BeanFactoryPostProcessor和BeanPostProcessor的注册，Spring的扫描就是通过 BenaFactoryPostProcessor来实现的，依赖注⼊就是通过BeanPostProcessor来实现的 8. 在Spring启动过程中还会去处理@Import等注解