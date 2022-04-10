



#### SpringBoot中常用注解及其底层实现

1. @SpringBootApplication注解：这个注解标识了⼀个SpringBoot⼯程，它实际上是另外三个注解 的组合，这三个注解是：

   a. @SpringBootConfiguration：这个注解实际就是⼀个@Configuration，表示启动类也是⼀个 配置类

   b.@EnableAutoConfiguration：向Spring容器中导⼊了⼀个Selector，⽤来加载ClassPath下 SpringFactories中所定义的⾃动配置类，将这些⾃动加载为配置Bean

   c.@ComponentScan：标识扫描路径，因为默认是没有配置实际扫描路径，所以SpringBoot扫 描的路径是启动类所在的当前⽬录

2. @Bean注解：⽤来定义Bean，类似于XML中的标签，Spring在启动时，会对加了@Bean注 解的⽅法进⾏解析，将⽅法的名字做为beanName，并通过执⾏⽅法得到bean对象

3. @Controller（控制器层）、@Service（业务层）、@Component（组件）、@Respository（数据访问组件DAO）、@ResponseBody、@Autowired（按照类型自动注入）、@Resource（按照名称进行注入）

4. @Async异步线程，需要添加ThreadPoolConfig类并使用@EnableAsync注解开启

5. @Transaction事务注解





#### springboot获取入参

https://blog.csdn.net/weixin_51128948/article/details/115894071