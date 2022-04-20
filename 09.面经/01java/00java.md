#### 抽象类和接口的区别

含有abstract修饰符的class即为抽象类，abstract 类不能创建实例对象。含有abstract方法的类必须定义为abstract class，abstract class类中的方法不必是抽象的。abstract class类中定义抽象方法必须在具体(Concrete)子类中实现。所以，不能有抽象构造方法或抽象静态方法。如果子类没有实现抽象父类中的所有抽象方法，那么子类也必须定义为abstract类型。

接口（interface）可以说成是抽象类的一种特例，接口中的所有方法都必须是抽象的。接口中的方法定义默认为public abstract类型，接口中的成员变量类型默认为public static final。

下面比较一下两者的语法区别：

1.抽象类可以有构造方法，接口中不能有构造方法。

2.抽象类中可以有普通成员变量，接口中没有普通成员变量

3.抽象类中可以包含非抽象的普通方法，接口中的所有方法必须都是抽象的，不能有非抽象的普通方法。

4.抽象类中的抽象方法的访问类型可以是public，protected和（默认类型,虽然eclipse下不报错，但应该也不行），但接口中的抽象方法只能是public类型的，并且默认即为public abstract类型。

5.抽象类中可以包含静态方法，接口中不能包含静态方法

6.抽象类和接口中都可以包含静态成员变量，抽象类中的静态成员变量的访问类型可以任意，但接口中定义的变量只能是public static final类型，并且默认即为public static final类型。

7.一个类可以实现多个接口，但只能继承一个抽象类。

下面接着再说说两者在应用上的区别：

接口更多的是在系统架构设计方法发挥作用，主要用于定义模块之间的通信契约。而抽象类在代码实现方面发挥作用，可以实现代码的重用，例如，模板方法设计模式是抽象类的一个典型应用，假设某个项目的所有Servlet类都要用相同的方式进行权限判断、记录访问日志和处理异常，那么就可以定义一个抽象的基类，让所有的Servlet都继承这个抽象基类，在抽象基类的service方法中完成权限判断、记录访问日志和处理异常的代码，在各个子类中只是完成各自的业务逻辑代码。



#### private、default、protected和public

private：只能当前类访问

default：当前类和同一个包

protected：子类、父类可以使用

public：所有类，不管在不在同一个包



#### 创建对象的有哪几种方式

- new关键字

- Class.newInstance

  ```java
  Person person = Person.class.newInstance();
  ```

- Constructor.newInstance

  ```java
  // 包括public的和非public的，当然也包括private的
  Constructor<?>[] declaredConstructors = Person.class.getDeclaredConstructors();
  // 只返回public的~~~~~~(返回结果是上面的子集)
  Constructor<?>[] constructors = Person.class.getConstructors();
  
  Constructor<?> noArgsConstructor = declaredConstructors[0];
  Constructor<?> haveArgsConstructor = declaredConstructors[1];
  
  noArgsConstructor.setAccessible(true); // 非public的构造必须设置true才能用于创建实例
  Object person1 = noArgsConstructor.newInstance();
  Object person2 = declaredConstructors[1].newInstance("fsx", 18);
  ```

- Clone方法

  ```java
  public class Person implements Cloneable {
  	// 访问权限写为public，并且返回值写为person
      @Override
      public Person clone() throws CloneNotSupportedException {
          return (Person) super.clone();
      }
  }
  public class Main {
      public static void main(String[] args) throws Exception {
          Person person = new Person("fsx", 18);
          Object clone = person.clone();
          System.out.println(person);
          System.out.println(clone);
          System.out.println(person == clone); //false
      }
  }
  ```

- 反序列化

```java
public class Main {
    public static void main(String[] args) throws Exception {
        Person person = new Person("fsx", 18);
        byte[] bytes = SerializationUtils.serialize(person);
        // 字节数组：可以来自网络、可以来自文件（本处直接本地模拟）
        Object deserPerson = SerializationUtils.deserialize(bytes);
        System.out.println(person);
        System.out.println(deserPerson);
        System.out.println(person == deserPerson);
    }
}
```



#### 反射获取类的三种方式

- 通过类全限定名进行获取Class.forName("classFullName");

```java
Class c=Class.forName("com.mysql.jdbc.Driver");
```

- 通过类型获取，另外任何数据类型都有一个静态的属性class

```java
//类型获取
Class c=StudentInfo.class;
//任何类都有class属性
Class booleanClass = boolean.class;
Class integerClass = int.class;
```

- 借助对象，使用getClass()获取

```java
StudentInfo s=new StudentInfo("Jack", 23);
Class c=s.getClass();
```



#### 深拷贝和浅拷贝

浅拷贝：简单类型进行值传递，引用数据类型进行引用传递

深拷贝：简单类型进行值传递，引用数据类型重新创建对应的对象并进行拷贝

简答来说就是修改浅拷贝的引用数据类型的数据，原来的对象也会改变。但是深拷贝则不会。



#### Exception与Error包结构。OOM你遇到过哪些情况，SOF你遇到过哪些情况

java将可抛出（Throwable）的结构分为三类：被检查的异常（Checked Exception），运行时异常（RuntimeException），错误（Error）。

- 运行时异常RuntimeException
  - 定义：RuntimeException及子类都被称为运行时异常， 特点：java编译器不会检查它，也就是说当程序中可能出现这类异常时，倘若既“没有通过throws声明抛出它”，也“没有用try-catch语句捕获它”，还是会编译通过。
  - 例如，除数为0时产生的ArithmeticException异常，数组越界时产生的IndexOutOfBoundsException异常，fail-fail机制产生的ConcurrentModificationException异常等，都属于运行时异常。

- 堆内存溢出OutOfMemoryError（OMM）
  - 除了程序计数器外，虚拟机内存的其他几个运行时区域都有发生OutOfMemoryError（OMM）异常的可能。
  - Java Heap溢出，一般的异常信息：java.lang.OutOfMemoryError：Java heap spacess。 java堆用于存储对象实例，我们只要不断的创建对象，并保证GC Roots到对象之间有可达路径来避免垃圾回收机制清除这些对象，就会在对象数量达到最大堆容量限制后产生内存溢出异常。

- 堆栈溢出 StackOverflow（SOF）
  - StackOverflowError的定义：当应用程序递归太深而发生堆栈溢出时，抛出该错误。因为栈一般默认为1-2m，一旦出现死循环或者是大量的递归调用，在不断的压栈过程中，造成栈容量超过1m而导致溢出。
  - 栈溢出的原因：递归调用。大量循环或死循环。全局变量是否过多。数组、List、map数据过大



#### 在 Java 中定义一个不做事且没有参数的构造方法有什么作用？

在字类没有用super()来调用父类的时候，在创建字类对象的时候会调用父类中“没有参数的构造方法”。如果不声明，则会报错。

解决方法是写一个空的无参构造，或者直接用super()来调用构造方法。



#### jar包和war包有什么区别？

jar包可以单独在jvm中运行，可以被其他项目引入调用，war包则用于web服务，需要tomcat容器运行



#### final关键字，（String是final修饰的）

- 修饰类，无法被继承
- 修饰方法，无法被重写
- 修饰变量，只能赋值一次



##### 最有效率计算2*8：*

##### 2<<n，左移运算符相当于*2的n次
