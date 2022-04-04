在Util工具类中的一个方法里,需要调用mapper来实现功能,在静态方法里直接注入@Autowired会报NullPointException空指针异常.

尝试过两种方法实现功能:

### 方法一

------

1.类上添加@Service注解

2.创建静态mapper对象

3.@Autowired注入set方法,调用静态对象

```java
@Service
public class CommonUtil{
    private static CommonMapper commonMapper;
    
    @Autowired
    public void setCommonMapper(CommonMapper commonMapper){
        CommonUtil.commonMapper = commonMapper;
    }
}
```

此方法可以正常运行,不会报空指针异常,但Sonar扫描时会提示:普通方法调用静态字段问题

建议修改方法:将非静态方法中对静态方法赋值的语句,单独封装一个静态方法,并且加上synchronized关键字,这样就不会导致多线程修改字段导致的其他问题.

### 方法二

------

1.类上添加@Component注解

2.Autowired注入需要使用的mapper对象(非静态的)

3.添加一个本类类型的静态字段

4.创建初始化方法,贴上@PostConstruct标签,用于注入bean

5.创建方法调用mapper

```java
@Component
public class CommonUtil{
    @Autowired
    private CommonMapper commonMapper;
    
    private static CommonUtil commonUtil;
    
    @PostConstruct
    public void init(){
        checkUtil = this;
        checkUtil.commonMapper = this.commonMapper;
    }
}
```

方法中调用checkUtil.commonMapper即可