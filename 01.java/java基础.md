#### Java特性：

- **跨平台：** Java程序不是运行在操作系统上的，而是运行在JVM中，编译成字节码文件，进行运行
- **面向对象：** 封装，继承，多态
- **多线程**
- **多态**

#### 基础类型和所占字节

| 类型    | 所占字节                                                | 对应的包装类 |
| ------- | ------------------------------------------------------- | ------------ |
| byte    | 1                                                       |              |
| short   | 2                                                       |              |
| int     | 4                                                       | Integer      |
| long    | 8                                                       |              |
| char    | 2（C语言中是1字节）可以存储一个汉字                     | Character    |
| float   | 4                                                       |              |
| double  | 8                                                       |              |
| boolean | false/true(理论上占用1bit,1/8字节，实际处理按1byte处理) |              |

#### final、finally、finalize区别

- final用于声明属性，方法和类，分别表示属性不可交变，方法不可覆盖，类不可继承（类中的方法默认final）。
- finally是异常处理语句结构的一部分，表示总是执行。
- finalize是Object类的一个方法，在垃圾收集器执行的时候会调用被回收对象的此方法，供垃圾收集时的其他资源回收，例如关闭文件等。

#### static 静态成员 / 静态内部类与非静态内部类

#### 重写和重载

- 重写（Override）是父类与子类之间的多态性，实质是对父类的函数进行重新定义，如果在子类中定义某方法与其父类有相同的名称和参数则该方法被重写，不过子类函数的访问修饰权限不能小于父类的；若子类中的方法与父类中的某一方法具有相同的方法名、返回类型和参数表，则新方法将覆盖原有的方法，如需父类中原有的方法则可使用 super 关键字
- 重载（Overload）是让类以统一的方式处理不同类型数据的一种手段，实质表现就是多个具有不同的参数个数或者类型的同名函数（返回值类型可随意，不能以返回类型作为重载函数的区分标准）同时存在于同一个类中，是一个类中多态性的一种表现（调用方法时通过传递不同参数个数和参数类型来决定具体使用哪个方法的多态性）。

#### equals

- equals()比较实例中所包含的内容是否相同。

- ==用来比较两个地址是否相同

#### 值传递和引用传递

- 基本数据类型传值，对形参的修改不会影响实参；

- 引用类型传引用，形参和实参指向同一个内存地址（同一个对象），所以对参数的修改会影响到实际的对象。
- String, Integer, Double等immutable的类型特殊处理，可以理解为传值，最后的操作不会修改实参对象。

#### JDK , JRE , JVM 之间的关系

- JDK（Java Development Kit）Java开发工具包
- JRE（Java Runtime Environment）：Java运行环境
- JVM（Java Virtual Machine）：Java虚拟机

#### Object类中的方法

```java
registerNatives()   //私有方法
getClass()    //返回此 Object 的运行类。
hashCode()    //用于获取对象的哈希值。
equals(Object obj)     //用于确认两个对象是否“相同”。
clone()    //创建并返回此对象的一个副本。 
toString()   //返回该对象的字符串表示。   
notify()    //唤醒在此对象监视器上等待的单个线程。   
notifyAll()     //唤醒在此对象监视器上等待的所有线程。   
wait(long timeout)    //在其他线程调用此对象的 notify() 方法或 notifyAll() 方法，或        者超过指定的时间量前，导致当前线程等待。   
wait(long timeout, int nanos)    //在其他线程调用此对象的 notify() 方法或 notifyAll() 方法，或者其他某个线程中断当前线程，或者已超过某个实际时间量前，导致当前线程等待。
wait()    //用于让当前线程失去操作权限，当前线程进入等待序列
finalize()    //当垃圾回收器确定不存在对该对象的更多引用时，由对象的垃圾回收器调用此方法。
```

## JAVA IO

字节流和字符流

### BIO和NIO

**BIO**
       同步阻塞式IO，服务器实现模式为一个连接一个线程，即客户端有连接请求时服务器端就需要启动一个线程进行处理，如果这个连接不做任何事情会造成不必要的线程开销，当然可以通过线程池机制改善

**NIO**
       同步非阻塞式IO，服务器实现模式为一个请求一个线程，即客户端发送的连接请求都会注册到多路复用器上，多路复用器轮询到连接有I/O请求时才启动一个线程进行处理。

## 反射

**反射的实现**

- Object ——> getClass();
- 任何数据类型（包括基本数据类型）都有一个“静态”的class属性
- 通过Class类的静态方法：forName（String className）(常用)

**应用场景：**

**获取反射的方法**

**安全隐患**

## String

**字符串常量池**

  字符串是基本的引用类型。频繁的创建消耗性能，创建字符串时候，首先认为字符串在常量池中，如果存在，返回引用实例，如果不存在，则创建并存入常量池。字符串常量池存在于方法区
　　String str1 = new String(“A”+“B”) ; 会创建多少个对象?
　　String str2 = new String(“ABC”) + “ABC” ; 会创建多少个对象?

str1：
　　字符串常量池：“A”,“B”,“AB” : 3个
　　堆：new String(“AB”) ：1个
　　引用：str1 ：1个
　　总共 ：5个

str2 ：
　　字符串常量池：“ABC” : 1个
　　堆：new String(“ABC”) ：1个
　　引用：str2 ：1个
　　总共 ：3个

**特性：**

**常用方法**

```java
String intern()  		//如果String采用new的方法创建，不会存入常量池，inter()方法可以手工的加入到常量池
int length();			//返回字符串长度
char chatAt(int a);		//返回指定的字符
char  toCharArray();	//返回char类型的数组，将String转为char[]
int indexOf(char a);	//返回字符首次出现的位置下标
int lastIndexOf(char a) //得到指定内容最后一次出现的下标
toUpperCase();  toLowerCase();	//字符串大小写的转换
String[] split(char a);			//拆分，返回String类型的数组
boolean equals(Object anObject)		//比较是否相等
trim();　　				//去掉字符串左右空格　　
replace(char oldChar,char newChar);		//新字符替换旧字符，也可以达到去空格的效果一种
String substring(int beginIndex,int endIndex)　　//截取字符串
boolean equalsIgnoreCase(String) 		//忽略大小写的比较两个字符串的值是否一模一样
boolean contains(String) 		//判断一个字符串里面是否包含指定的内容
boolean startsWith(String)　　//测试此字符串是否以指定的前缀开始
boolean endsWith(String)　　//测试此字符串是否以指定的后缀结束
String replaceAll(String,String) //将某个内容全部替换成指定内容
String repalceFirst(String,String) //将第一次出现的某个内容替换成指定的内容
```

**StringBuffer, StringBuilder**

- StringBuffer（字符串变量，线程安全，不能赋空值）
- StringBuilder（字符串变量，非线程安全）
  　　StringBuffer会把字符串当作一个变量来操作，相对于经常修改的字符串，推荐使用StringBuffer类型的变量，因为不用创建新的变量，所以对于下面的代码，StringBuffer比String会快速非常多；

```java
String str1="123";     str1="234";
StringBuffer str2="123"; 	str2="234";
```

但是下面这种情况就是例外了

```java
String S1 = “This is only a” + “ simple” + “ test”;
StringBuffer Sb = new StringBuilder(“This is only a”).append(“ simple”).append(“ test”);
```

​	这是因为S1创建的时候，jvm会进行优化 String S1 = “This is only a simple test”;而StringBuffer需要修改两次，所以会慢很多。

​	java.lang.StringBuilder一个可变的字符序列是5.0新增的。此类提供一个与 StringBuffer 兼容的 API，但不保证同步。该类被设计用作 StringBuffer 的一个简易替换，用在字符串缓冲区被单个线程使用的时候（这种情况很普遍）。如果可能，建议优先采用该类，因为在大多数实现中，它比 StringBuffer 要快。两者的方法基本相同。

# Comparable 和 Comparator

Comparable是一个排序接口，List和Array实现之后可以直接调用sort进行排序
Comparator是比较器接口，大于返回正数，等于为0，小于为负数

```java
public class test {
    public static void main(String[] args) {
        List<Student> studentList = Arrays.asList(new Student("liming", 90),
                new Student("xiaohong", 95),
                new Student("zhoubin", 88),
                new Student("xiaoli", 94)
        );
        // 排序前
        System.out.println(studentList);
        Collections.sort(studentList);
        // 排序后
        System.out.println(studentList);
        for(Student student : studentList){
            System.out.println(student.equals(new Student("xiaohong", 95)));
        }
    }
}
class Student implements Comparable<Student>{
    String name;
    int record;
    public Student(){}
    public Student(String name,int record){
        this.name = name;
        this.record = record;
    }
    public boolean equals(Student student) {
        // 拿名字和成绩进行对比
        return name.equals(student.name)
                && record == student.record;
    }
    public int compareTo(Student stu) {
        // 调用String 类的compareTo方法，返回值 -1，0，1
        return this.name.compareTo(stu.name);
    }
    public String toString() {
        return "Student{name="+ name + ", record=" + record +'}';
    }
}
```

区别：
　　1.Comparable接口位于java.lang包下；Comparator位于java.util包下
　　2.Comparable接口只提供了一个compareTo()方法；Comparator接口不仅提供了compara()方法，还提供了其他默认方法，如reversed()、thenComparing()，使我们可以按照更多的方式进行排序
　　3.如果要用Comparable接口，则必须实现这个接口，并重写comparaTo()方法；但是Comparator接口可以在类外部使用，通过将该接口的一个匿名类对象当做参数传递给Collections.sort()方法或者Arrays.sort()方法实现排序。Comparator体现了一种策略模式，即可以不用要把比较方法嵌入到类中，而是可以单独在类外部使用，这样我们就可有不用改变类本身的代码而实现对类对象进行排序。

java多态原理
类为什么是单继承的
Serializable 序列话和反序列化 ，为什么要序列化
Collection
 List
 迭代器 Iterator
 ArrayList 和 LinkedList 的区别和使用场景
 Arraylist 的扩容
 ArrayList 和 Vector
 多线程场景下的 ArrayList
 为什么 ArrayList 的 elementData 加上 transient 修饰？
 Set
 List 和 Set 的区别
 HashSet 的实现原理，如何保证数据不重复
 HashSet 和 HashMap 的区别
 TreeSet 和 HashSet
 Queue
 接口实现类
 Deque
 PriorityQueue 实现大顶堆和小顶堆

Map
 HashMap
 1.7 / 1.8 版本实现原理及比较
 put 的流程
 扩容实现
 解决哈希冲突 （ 其他解决哈希冲突的方法）
 长度为什么是 2的次幂
 什么类适合作为 key
 ConcurrentHashMap
 1.7/1.8 版本实现原理
 是否可以存 null 值，为什么
 跟 HashTable 的区别
 TreeMap
 跟HashMap的区别

Throwable
 Error 和 Exception
 throw 和 throws
 运行时异常
 编译时异常
 JVM 如何处理异常
 常见的 RuntimeException
 try-catch-finally 的执行