# junit + powerMockit +assertJ单元测试

**单元测试的目的：**

- 单元测试不但会使你的工作完成得更轻松。而且会令你的设计会变得更好，甚至大大减少你花在调试上面的时间

- 提高代码质量 

- 减少bug，快速定位bug 

- 放心地修改、重构

**单元测试的编写原则：**

- 一个测试单元应只关注**一个功能函数**，证明它是正确的；
- 每个测试单元必须完全独立、能单独运行。
- **测试代码要能够快速执行。**
- 不能为了单元测试而修改已完成的代码；
- 在编写代码后执行针对本次的单元测试，并执行之前的单元测试用例。以保证你后来编写的代码不会破坏任何事情；
- **如果一个函数复杂到无法单测，那就说明模块的抽象有问题**















































再次强调下，单元测试应该在编码的层级之上，这个意思代表了编码应该根据单元测试来编写，而不是我们的测试根据编码来完成，如果很难理解，则试着体会下如下两个情景

- 单元测试现在通过了，但是代码由别人更改的时候新增或者删除了逻辑，但是针对单元测试来说，一般满足一个Test的功能都是重要分支，重要分支操作失败的时候会报错，这个时候就会提醒当前编写代码的人员，是否少考虑的该种情况
- 单元测试用例是一个中途推出的测试，那么是否需要把不会进行的方法进行覆盖呢：答案是需要，单元测试在编码之上，我们现在可以保证逻辑为中途推出，但是很难保证以后别人更改代码的时候不能充分理解逻辑，导致代码不会退出，而是继续执行下去了，所以一定要对所有的方法进行打桩



# 单元测试编写

- 单元测试代码位置应该和被测试的路径保持一致（不一致有时会导致无法正确进行覆盖率的计算，甚至报错）
- 单元测试类名为：被测试类名Test.java
- Maven依赖

```
<dependency> 
    <groupId>org.springframework.boot</groupId> 
    <artifactId>spring-boot-starter-test</artifactId> 	
    <scope>test</scope> 
</dependency>
<dependency>
    <groupId>org.powermock</groupId>
    <artifactId>powermock-api-mockito2</artifactId>
    <version>2.0.0</version>
    <scope>test</scope>
    </dependency>
<dependency>
<groupId>org.powermock</groupId>
    <artifactId>powermock-module-junit4</artifactId>
    <version>2.0.0</version>
    <scope>test</scope>
</dependency>
```

- Mockito参数匹配器（Matcher）

```
任意函数：
    Mockito.any(): 匹配任意参数，可以匹配null，集合、数字、map、字符串等 任何类型;
    Mockito.anyString()：匹配任意字符串,不包括null;
    Mockito.anyLong(): 匹配任意长数字,不包括null;
    Mockito.anyInteger()：匹配任意数字,不包括null;
    Mockito.anyDouble(): 匹配任意双精度，不包括null;
    Mockito.anyFloat(): 匹配任意浮点数字，不包括null;
    Mockito.anyBoolean(): 匹配任意布尔值，不包括null;
    Mockito.anyList(): 匹配任意集合，不包括null;
等同：
	Mockito.eq(): 匹配等于传入值的参数，包括任意类型的数字、字符串、布尔值、java类型等
正则匹配：
	Mockito.matches():匹配传入的正则表达式
```

- PowerMockit外部依赖模拟

```
PowerMockit要求测试类已PowerMockRunner.class启动，添加启动类的注解：
@RunWith(PowerMockRunner.class) 
```

- 其他

```
@Before : 被@Before修饰的方法会在每个测试方法之前执行。可以将公共依赖的代码放在该方法中。 
@After: 被@After修饰的方法会在每个测试方法之后执行。可以将公共断言代码放在该方法中。
@BeforeClass : 被@BeforeClass 修饰的方法每个类会执行一次。
@AfterClass: 被@After修饰的方法会在所有的测试用例执行完之后执行一次。
```

在Mockito中打桩（即stub)有两种方法when(...).thenReturn(...)和doReturn(...).when(...)。这两个方法在大部分情况下都是可以相互替换的，但是在使用了Spies对象（@Spy注解），而不是mock对象（@Mock注解)的情况下他们调用的结果是不相同的（目前我只知道这一种情况，可能还有别的情形下是不能相互替换的）。
 when(...) thenReturn(...)不会调用真实方法
 doReturn(...).when(...) 会调用真实的方法，如果你不想调用真实的方法而是想要mock的话，就不要使用这个方法

## 开始编写

- 测试方法注释

  ```
  /**
  * 测试方法注释
  * 
  * @case 单元测试情况
  * @expect 希望的结果，断言
  */
  ```

- 测试方法名称为：被测试方法Test(序号) 的方式进行命名

  例：对于方法getServerityDegree()方法需要两个单元测试的方法进行覆盖，则两个单元测试的方法为getServerityDegreeTest1()和getServerityDegreeTest2()。

- 编写流程

![](https://youcai922.github.io/99.src/img/测试方法编写流程.png)

- 被测类实体及依赖注入

```java
//被测类需要使用InjectMocks，其余调用的dao层则使用Mock
@InjectMocks
MaterialServiceImpl materialService;

@Mock
MaterialConfigDao materialConfigDao;

@Mock
ProductDao productDao;

@Mock
MaterialMapConfigDao materialmapconfigDao;

@Mock
ProductAttributeDao productAttributeDao;

@Mock
MaterialaggrDataDaoImpl materialaggrDataDao;

@Mock
DeviceDataService deviceDataService;
```

- 打桩

**什么是打桩？**

项目中，有些函数需要处理某个服务的返回结果，而在对函数单元测试的时候，又不能启动那些服务，有些函数会去插入或者删除数据库，但是我们又不能实际去删除或者插入业务数据库，这里就可以利用Mockito工具进行打桩。

**Mockito工具有几种打桩方式？**

Mockito中的Mock和Spy都可用于拦截那些尚未实现或不期望被真实调用的对象和方法，并为其设置自定义行为。二者的区别在于：

1、Mock声明的对象，对函数的调用均执行mock（即虚假函数），不执行真正部分。

2、Spy声明的对象，对函数的调用均执行真正部分。但是返回的结果和真实执行的部分无关。

```java
//普通打桩
//参数匹配时，一旦函数参数里边有任何一个any或类似的Matcher函数（如anyInt，anyString等）时，其他所有参数也必须以同样的形式出现
@Test 
public void demoTest1(){ 
	File file = PowerMockito.mock(File.class); 
	FlySunDemo demo = new FlySunDemo(); 
	PowerMockito.when(file.exists(Mockito.any())).thenReturn(true); 	Assert.assertThat(demo.callArgumentInstance(file)).isEqualTo(true); 
} 
//打桩静态方法、被final修饰的方法
//静态方法一般可以不进行打桩，因为静态方法一般用来处理数据或者作为工具类使用，如果入参是合理的，可以使用源代码测试
@Test 
@PrepareForTest(File.class)
	public void demoTest2(){ 
	PowerMockito.mockStatic(File.class)
	PowerMockito.when(File.exists(Mockito.any())).thenReturn(true); 
	Assert.assertThat(demo.callArgumentInstance(file)).isEqualTo(true); 
} 
//打桩私有方法
@Test 
@PrepareForTest(File.class)
public void demoTest1(){ 
	File file = PowerMockito.mock(File.class); 
	FlySunDemo demo = new FlySunDemo(); 
	PowerMockito.when(file,"exists",Mockito.any()).thenReturn(true);
    Assert.assertThat(demo.callArgumentInstance(file)).isEqualTo(true); 
} 
//打桩无返回值的方法
@Test 
public void demoTest1(){ 
	File file = PowerMockito.mock(File.class); 
	FlySunDemo demo = new FlySunDemo(); 
	PowerMockito.doAnswer(new Answer<Object>() {  
        	public Object answer(InvocationOnMock invocation) {  
            		Object[] args = invocation.getArguments();  
            		return "called with arguments: " + args;  
        	}  
    	}).when(file).exists(Mockito.any());
	Assert.assertThat(demo.callArgumentInstance(file)).isEqualTo(true); 
} 
//其他打桩方式
PowerMockito.doNothing().when();
PowerMockito.doThrow().when();
//注意：doNothing、doAnswer都会真实去调用被测方法，如果不期望去调用被测方法，可以使用如下：
PowerMockito.suppress(MemberMatcher.method(File .class,"exists"));


//模拟抛出异常：
@Test(expected = CommonManagerException.class)
public void demoTest1(){ 
	File file = PowerMockito.mock(File.class); 
	FlySunDemo demo = new FlySunDemo(); 
	Assert.assertThat(demo.callArgumentInstance(file)).isEqualTo(true); 
} 
//模拟依赖方法未抛出异常：
@Test(expected = CommonManagerException.class)
public void demoTest1(){ 
	File file = PowerMockito.mock(File.class); 
	PowerMockito.doNothing().doThrow(Exception.class);
	file.exists(Mockito.any())	
	Assert.assertThat(demo.callArgumentInstance(file)).isEqualTo(true); 
} 
//模拟被测试类的私有方法：
//测试方法不能有参数，除非进行参数测试
@Test
public void materialDosageTest() throws Exception {
    //1、构建测试参数
	Long startTime = TimeUtil.getFirstDayOfThisMonth(TimeUtil.localDateTime2timestamp(LocalDateTime.now()));
	//2、分析、模拟外部依赖(调用其他类、方法都算外部依赖)
	//2.1 外部bean调用模拟
	MaterialConfig materialConfig1 = new MaterialConfig();
	List<MaterialConfig> materialConfigList = Arrays.asList(materialConfig1);
	PowerMockito.when(materialConfigDao.queryAllMaterialConfigList()).thenReturn(materialConfigList);
	//2.3本类的其他私有方法调用
	MaterialServiceImpl spy = PowerMockito.spy(materialService);
	PowerMockito.doNothing().when(spy, "getMaterialAggrDataByProcedure", Mockito.anyList(), Mockito.anyLong());
	//3、调用被测方法
	spy.materialDosage(startTime);
	//4、断言测试结果
	PowerMockito.verifyPrivate(spy).invoke("getMaterialAggrDataByProcedure",Mockito.anyList(),Mockito.anyLong());
}
//由于被测类已经使用了@Mock打桩，我们在调用被测类的方法时，需要重新打桩某一个方法，使用spy
```

- 断言 org.assertj.core.api.Assertions.assertThat

  - 断言空

    - assertThat(var).isNull(); 

  - 断言字符串

    断言 是注解 		assertThat(A.class).isAnnotation(); 

    断言 不是注解 	assertThat(A.class).isNotAnnotation(); 

    断言 存在注解 	assertThat(A.class).hasAnnotation(Magical.class); 

    断言 不是接口 	assertThat(A.class).isNotInterface(); 

    断言 是否为指定Class实例 	assertThat("string").isInstanceOf(String.class); 

    断言 类是给定类的父类 	assertThat(A.class).isAssignableFrom(Employee.class); 

  - 断言数字：

    断言相等 		assertThat(num).isEqualTo(42); 

    断言大于 大于等于 

    ​	assertThat(num).isGreaterThan(38)

    ​	assertThat(num).isGreaterThanOrEqualTo(38); 

    断言小于 小于等于

    ​	assertThat(num).isLessThan(58)

    ​	assertThat(num).isLessThanOrEqualTo(58); 

    断言0 	assertThat(num).isZero(); 

    断言正数、非负数

    ​	assertThat(num).isPositive()

    ​	assertThat(num).isNotNegative(); 

    断言负数、非正数 

    ​	assertThat(num).isNegative()

    ​	assertThat(num).isNotPositive();

  - 断言日期：

    断言与指定日期相同 不相同 在指定日期之后 在指定日期之前

    ​	assertThat(dateStr).isEqualTo("2014-02-01").

    ​	assertThat(dateStr).isNotEqualTo("2014-01-01") 

    ​	assertThat(dateStr).isAfter("2014-01-01")

    ​	assertThat(dateStr).isBefore("2014-03-01"); 

    断言两时间相差100毫秒 	assertThat(date1).isCloseTo(date2, 100); 

    断言日期忽略毫秒后，与给定的日期相等 	assertThat(date1).isEqualToIgnoringMillis(date2); 

    断言日期与给定的日期具有相同的年月日时分秒 	assertThat(date1).isInSameSecondAs(date2); 

    断言 日期忽略秒，与给定的日期时间相等 	assertThat(date1).isEqualToIgnoringSeconds(date2); 

    断言 日期与给定的日期具有相同的年月日时分 	assertThat(date1).isInSameMinuteAs(date2); 

    断言 日期忽略分，与给定的日期时间相等 	assertThat(date1).isEqualToIgnoringMinutes(date2); 

    断言 日期与给定的日期具有相同的年月日时 	assertThat(date1).isInSameHourAs(date2); 

    断言 日期忽略小时，与给定的日期时间相等 	assertThat(date1).isEqualToIgnoringHours(date2); 

    断言 日期与给定的日期具有相同的年月日 	assertThat(date1).isInSameDayAs(date2); 

  - 断言集合：

    断言 列表是空的 	assertThat(newArrayList()).isEmpty(); 

    断言 列表的开始 结束元素 	assertThat(newArrayList(1,2,3)).startsWith(1).endsWith(3); 

    断言 列表包含元素 并且是排序的 	assertThat(newArrayList(1,2,3)).contains(1,atIndex(0)).contains(2).isSorted();

    断言 被包含与给定列表 	assertThat(newArrayList(3,1,2)).isSubsetOf(newArrayList(1,2,3,4)); 

    断言 存在唯一元素 	assertThat(Lists.newArrayList("a","b","c")).containsOnlyOnce("a"); 

    断言map：

    断言 map 不为空 size	assertThat(foo).isNotEmpty().hasSize(3); 

    断言 map 包含元素 	assertThat(foo).contains(entry("A", 1), entry("B", 2)); 

    断言 map 包含key 	assertThat(foo).containsKeys("A", "B", "C"); 

    断言 map 包含value	assertThat(foo).containsValue(3); 

  - 断言类：

    断言 注解类	assertThat(Magical.class).isAnnotation(); 

    断言 不是注解 	assertThat(Ring.class).isNotAnnotation(); 

    断言 存在注解 	assertThat(Ring.class).hasAnnotation(Magical.class); 

    断言 不是接口 	assertThat(Ring.class).isNotInterface(); 

    断言 是否为指定Class实例 	assertThat("string").isInstanceOf(String.class); 

    断言 类是给定类的父类 	assertThat(Person.class).isAssignableFrom(Employee.class); 