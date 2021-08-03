### Java Lambda表达式

Lambda表达式：一段可以传递的代码，因此它可以被执行一次或者多次。

语法格式：(parameters) -> expression 或 (parameters) ->{ statements; }

lambda表达式重要特征：

- **可选类型声明：**不需要声明参数类型，编译器可以统一识别参数值。
- **可选的参数圆括号：**一个参数无需定义圆括号，但多个参数需要定义圆括号。
- **可选的大括号：**如果主体包含了一个语句，就不需要使用大括号。
- **可选的返回关键字：**如果主体只有一个表达式返回值则编译器会自动返回值，大括号需要指定明表达式返回了一个数值。

lambda表达式实例

```java
//不需要参数，直接返回5
()->5
//接受一个参数(数字类型)，返回其2倍的值
x->2*x
//接收两个参数，并且返回他们的和
(x,y)->x+y
//接收两个int类型的整数，返回他们的和
(int x,int y)->x+y
//接收String对象，输出他们，不返回值
(String s)->System.out.print(s)
```

```java
public class Java8Tester {
   public static void main(String args[]){
      Java8Tester tester = new Java8Tester();
        
      // 类型声明
      MathOperation addition = (int a, int b) -> a + b;
        
      // 不用类型声明
      MathOperation subtraction = (a, b) -> a - b;
        
      // 大括号中的返回语句
      MathOperation multiplication = (int a, int b) -> { return a * b; };
        
      // 没有大括号及返回语句
      MathOperation division = (int a, int b) -> a / b;
        
      System.out.println("10 + 5 = " + tester.operate(10, 5, addition));
      System.out.println("10 - 5 = " + tester.operate(10, 5, subtraction));
      System.out.println("10 x 5 = " + tester.operate(10, 5, multiplication));
      System.out.println("10 / 5 = " + tester.operate(10, 5, division));
        
      // 不用括号
      GreetingService greetService1 = message ->
      System.out.println("Hello " + message);
        
      // 用括号
      GreetingService greetService2 = (message) ->
      System.out.println("Hello " + message);
        
      greetService1.sayMessage("Runoob");
      greetService2.sayMessage("Google");
   }
    
   interface MathOperation {
      int operation(int a, int b);
   }
    
   interface GreetingService {
      void sayMessage(String message);
   }
    
   private int operate(int a, int b, MathOperation mathOperation){
      return mathOperation.operation(a, b);
   }
}
```



```java
Lambda排序
String[] strings={"aa","aaa","a","aaaa"};
Comparator<String> comp = (String first,String second)->Integer.compare(first.length(),second.length);
Arrays.sort(strings,comp);

Comparator<String> comp=(String first,String second)->{
    if(first.length()<second.length())return -1;
    else if(first.length()>second.length())return 1;
    else return 0;
}
```

