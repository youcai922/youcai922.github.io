# 代码阅读类

####  下面的代码创建了几个对象

```
String a = new String("123");
```

​	两个，a会先在常量池创建“123”字符串常量，当new的时候就会在堆内存中创建一个对象，此时会把常量池中的字符串常量拷贝一份副本到给到堆内存中的对象，堆内存中的这个对象就会把地址值赋给a。常量池中对象的地址值和堆内存中对象的地址值是不一样的，a指向的是堆内存中的对象，不是常量池中的对象。此时堆内存中有一个对象，常量池中有一个对象，所以创建了2个对象。

#### 判断是否相等

```java
String a="123";
String b="123";

true;
```

```java
String a=new String("123");
String b="123";

false;
```

```java
String a="1"+"2"+"3";
String b="123";

true;
//java有常量优化机制，编译的时候其实是a=“123”
```

```java
String a="12";
String b="123"
String c=a+"3";

a==c;false;
//a不是常量，所以不能用常量优化机制
```





#### String、StringBuffer、StringBuilder的区别

- String是final修饰的，不可变，如果修改，会新生成一个字符串对象，StringBuffer和StringBuilder是可变的
- StringBuffer是线程安全的、StringBuilder是线程不安全的，单线程环境下StringBuilder效率更高

