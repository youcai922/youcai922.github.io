Option类是为了解决Java空指针的异常引入的，Optional是一个容器类，可以保存类型T的值，代表这个值存在。或者仅仅保存null，标值这个值不存在。原来用null表示一个值不存在，现在Optional可以更好的表达这个概念。并且可以避免空指针异常。

- 创建Optional类对象的方法：
  - Optional.of(T t)：创建一个Optional实例，t必须为空
  - Optional.empty()：创建一个空的Opitonal实例
  - Optional。ofNullable(T t)：t可以为null
- 判断Optional容器中是否包含对象
  - bollean isPresent()：判断是否包含对象
  - void ifPresent(Consumer<? super T> consumer)：如果有值，就执行Consumer接口的实现代码，并且该值会作为参数传给它
- 获取Optional容器的对象
  - T get()：如果调用对象包含值，返回该值，否则抛异常
  - T orElse(T other)：如果有值则将其返回，否则返回指定的other对象。
  - T orElseGet(Supplier<? extends T> other)：如果有值则将其返回，否则返回Supplier接口实现提供的对象。
  - T orElseThrow(Supplier<? extends X> exceptionSupplier)：如果有值则将其返回，否则抛出由Supplier接口实现提供的异常。

```java
@Data
@Configuration
@AllArgsConstructor
public static class Boy{
    String name;
    Girl girl;
    public Boy(Girl girl){
        this.girl=girl;
    }
    public Girl getGirl(){
        return this.girl;
    }
}
@Data
@Configuration
@AllArgsConstructor
public static class Girl{
    String name;
}
public static void main(String[] args){
    Boy boy = null;
    //boy = new Boy();
    //boy = new Boy(new Girl("苍老师"));
    String girlName = getGirlName(boy);
    System.out.println(girlName);

}
public static String getGirlName(Boy boy){
    Optional<Boy> boyOptional = Optional.ofNullable(boy);
    //此时的boy1一定非空
    Boy boy1 = boyOptional.orElse(new Boy(new Girl("迪丽热巴")));
    Girl girl = boy1.getGirl();

    Optional<Girl> girlOptional = Optional.ofNullable(girl);
    //girl1一定非空
    Girl girl1 = girlOptional.orElse(new Girl("古力娜扎"));

    return girl1.getName();
}
```

